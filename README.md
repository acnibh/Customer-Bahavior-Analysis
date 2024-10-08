# CUSTOMER BEHAVIOR ANALYSIS
## Introduction
A store who sell their products on the ecommmerce platform such as Tiki, Lazada, Shopee. The had just achieved 10 billions VND for gross revenue and they want to double gross revenue at 20 billions in the next month. This project used a part of data from their sample customer order and Google Analytics. I found that the store should focus on launching effectively campaigns on Google, Youtube, Facebook and their website in the mid-week and mid-month.

### Tool
- Python (import data, cleaning data, analysis): Pandas, Numpy, Mathplotlip
- Power BI (visualization)

### Dataset & Code
Datasets include Customer Order table and Customer Traffic table in July
- Customer Order [Download](https://docs.google.com/spreadsheets/d/1_bY_9TAUyJd0uEjMWHLwjmQLfmqhcaCD/edit?usp=sharing&ouid=107093883134897721167&rtpof=true&sd=true)
- Customer Traffic in Google Analytics [Download](https://drive.google.com/file/d/1aSECTEr_p0SlLyyn_1ZRRiXK6HTeiznL/view?usp=sharing)
- Google Colab [Code](https://colab.research.google.com/drive/1DDtk4AxpWD1J6D85EhfVEWNvqP9SQSZg?usp=sharing)


## Data Model 
![image](https://github.com/user-attachments/assets/07f70833-f017-4f47-8b69-59ff1ebe1888)

## Business Question
- What are the special features of customer behavior?
- How to achieve 20 billions VND in the following month?

## Data Cleaning 
### Check
```python
df_order.info()
```
![image](https://github.com/user-attachments/assets/55459483-90ea-47da-b943-a09f76052228)

- Convert
```python
from datetime import datetime
df_order['created_day'] = pd.to_datetime(df_order['created_day'])
```
![image](https://github.com/user-attachments/assets/9f6ce05e-f963-477c-87a5-4c385fea5fac)
- Remove Duplicates
```python
df_order.drop_duplicates(inplace = True)
df_order.info()
```
![image](https://github.com/user-attachments/assets/31920c85-30a6-4646-aa88-69e9b15ee6ac)

## Data Analysis
### Payment Method
```python
df_order['payment_method_cate'] = df_order['payment_method'].apply(lambda x: 'VN Airpay Ibanking' if 'VN Airpay Ibanking' in x
                                                                   else 'Cybersoure' if 'Cybe' in x
                                                                   else x
                                                                   )
df_method_cate = (
    df_order[df_order['order_status'] == 'COMPLETED']
    .groupby('payment_method_cate')
    .agg(
        total_tickets = ('order_id', 'nunique')
    ).reset_index().sort_values(by = 'total_tickets', ascending = False)
)
```
![image](https://github.com/user-attachments/assets/8a5fec75-907c-4b5b-9be8-ad395d2018fa)
87.61% of customers use the 'Cash on Delivery' method for payment when shopping on e-commerce platforms.

### Time series

```python
df_order['year'] = pd.to_datetime(df_order['created_day']).dt.year
df_order['month'] = pd.to_datetime(df_order['created_day']).dt.month
df_order['day_name'] = pd.to_datetime(df_order['created_day']).dt.day_name()
df_order['revenue'] = df_order['item_quantity'] * df_order['onsite_original_price']
df_day_name = (
    df_order[df_order['order_status'] == 'COMPLETED'].groupby('day_name').agg(
        total_orders = ('order_id', 'nunique'),
        revenue = ('revenue', 'sum')
    ).reset_index().sort_values(by = 'revenue', ascending = False)
)
df_day = (
    df_order[df_order['order_status'] == 'COMPLETED'].groupby('day').agg(
        total_orders = ('order_id', 'nunique')
    ).reset_index().sort_values(by = 'day', ascending = True)
)
```
![image](https://github.com/user-attachments/assets/c2d92b85-d4e6-41a4-b2e4-a85f500b0bfd)
Orders on Friday accounted for 43.6% of the total orders for the week, and a large number of orders were placed between July 12th and 14th. The other days did not see much change.

### Frequency & Anomaly behavior
```python
def cal_pro(x):
  return (x == True).sum()

df_index_success = (
    df_order[df_order['order_status'] == 'COMPLETED']
    .groupby('customer_unique_id')
    .agg(
        n_success = ('order_id', 'nunique'), # Do giải thuyết đặt ra là 1 order có nhiều món hàng thì sẽ tách ra nên sẽ dùng nunique thay vì count
        n_cate_item_success = ('order_id', 'count'),
        s_money = ('onsite_original_price', 'sum'), # Do shipping fee nó ko thuộc về cửa hàng nên ko đc tính do total charge
        n_days = ('created_day', 'nunique'),
        n_promotion = ('check_price', cal_pro),
        s_discount = ('discount_value', 'sum')
    ).reset_index()
)
df_index_cancel = (
    df_order[df_order['order_status'] == 'CANCELLED']
    .groupby('customer_unique_id')
    .agg(
        n_cancelled = ('order_id', 'nunique')
    ).reset_index()
)
df_customer_value = pd.merge(df_index_success, df_index_cancel, how = 'left', on ='customer_unique_id').fillna(0)
df_customer_value['n_total'] = df_customer_value['n_success'] + df_customer_value['n_cancelled']
df_customer_value['success_rate'] = df_customer_value['n_success']/df_customer_value['n_total']
df_customer_value['promotion_rate'] = df_customer_value['n_promotion']/df_customer_value['n_cate_item_success']
df_customer_value['discount_rate'] = df_customer_value['s_discount']/ df_customer_value['s_money']

```
### Anomaly Behavior

![image](https://github.com/user-attachments/assets/901eae2a-22c2-4ff7-99b4-0c29f315dc4a)

After investigating why the number of orders on Friday and the mid-month days was unusually high, it was found that four customers made large purchases on the three mid-month days. They might be resellers, but since the provided data is limited, their exact identities are still unknown.
![image](https://github.com/user-attachments/assets/415102e1-06e1-49ca-add8-6ffb55238c34)
![image](https://github.com/user-attachments/assets/35a612db-71c2-4241-98ca-83a57b1441ee)



90.89% of customers only purchase once, and the four customers with the unusual behavior mentioned above indicate that the store doesn't have many frequent buyers. 98.67% of customers used promotions when purchasing products, with most of the discounts ranging from 10% to 50%. This suggests that the store is running promotional campaigns to attract new customers on the e-commerce platform.


### Success Rate
![image](https://github.com/user-attachments/assets/0aeadf42-35e8-46e9-8fc2-6fec164aa566)
There is an 80.02% success rate for orders, and the reason may be that customers want to change the product or address after placing the order. Among the canceled orders, 82.4% were canceled only once, and these customers had a 50% success rate. This suggests that the cancellations were not due to any fault of the store.

### Google Analytics
```python
df_gg_ana['platform'] = df_gg_ana['Source / Medium'].apply(lambda x: re.split('/',x, 1)[0])
df_gg_fil_revenue = df_gg_ana[df_gg_ana['Revenue'] > 0]
df_gg_fil_revenue['platform2'] = df_gg_fil_revenue['platform'].apply(lambda x: "Google" if "oogle" in x
                                                                     else "Direct" if "direct" in x
                                                                     else "Youtube" if "youtube" in x
                                                                     else "Facebook" if "facebook" in x
                                                                     else "Newsletter" if "newsletter" in x
                                                                     else "Messenger" if "messenger" in x
                                                                     else "Yahoo" if "yahoo" in x
                                                                     else "Workplace" if "workplace" in x
                                                                     else "Bing" if "bing" in x
                                                                     else "Linhkiendoc" if "linhkiendoc" in x
                                                                     else "Vnexpress" if "vnexpress" in x
                                                                     else "Instagram" if "instagram" in x
                                                                     else "Duckduckgo" if "duckduckgo" in x
                                                                     else "Tinhte" if "tinhte" in x
                                                                     else "Getpocket" if "getpocket" in x
                                                                     else "News.zing" if "news.zing" in x
                                                                     else "Ecosia" if "ecosia" in x
                                                                     else "other"
                                                                     )
```
![image](https://github.com/user-attachments/assets/60c8289f-964f-4386-a5b8-3d8678977f96)


Data from Google Analytics shows that Google, YouTube, Facebook, and the store's website generate the highest revenue and order numbers. However, it also indicates that they have run many advertising campaigns on these platforms, especially on Google. But the Bounce Rate and Conversion Rate on YouTube are not effective, indicating that the advertising campaign on this platform is not attracting new customers. The Conversion Rate is very low across all platforms, suggesting that the marketing campaigns are ineffective, and most customers are driven by product discounts.

## Recommendation
- The store should focus on running advertising and content marketing campaigns that are appealing to customers. Utilizing short videos and content aligned with current trends could help increase attraction, as well as improve the Conversion Rate, Bounce Rate, and transactions on each platform.
- The focus should primarily be on major social media platforms like Google, Facebook, Tiktok, and YouTube for advertising, to reach the right potential and target customers for the product. Time and budget should be allocated to invest in promotional content on these key platforms.

