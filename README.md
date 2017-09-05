Description 

The dataset used for this project purposes consists of 3 million open source online grocery store orders from more than 200 thousands of users. The dataset was available in one of the Kaggle’s competitions named ‘Instacart Market Basket Analysis’. This competition challenged data miners from all over the world to answer to the following question: “Which products will an Instacart consumer purchase in his next basket?”. 

Market basket analysis is an important component of every retail company. Simple, yet powerful - MBA is an inexpensive technique to identify cross-sell opportunities and engage customers. At the same time, personalized recommendation systems differentiate companies from the competition and they can lead to competitive advantages. Moreover, recommendation systems are proven to improve user experience, to increase user traffic and the number of purchases and to encourage user engagement and satisfaction. This competition was an opportunity for us to expand our knowledge and to gain hands on experience on models and techniques used in the fields of basket analysis and recommendation systems.


```python
import numpy as np # linear algebra
import pandas as pd # data processing, CSV file I/O (e.g. pd.read_csv)
import matplotlib.pyplot as plt
import seaborn as sns

color = sns.color_palette()

%matplotlib inline

pd.options.mode.chained_assignment = None  # default='warn
```
Data

The dataset used for this project purposes consists of 3 million open source online grocery store orders from more than 200 thousands of users. For each user, it contains between 4 and 100 of their orders, with the sequence of products purchased in each order. It also includes information concerning the week and hour of day that the order was placed, and a relative measure of time between orders. 

The dataset was too large to be handled and in order to deal with the memory overload problem, we kept the last 6 orders of every user and dropped the rest. We then split the dataset into 2 sets: train set and test set. The test set contained the last order of every user and the train set contained the rest. 


```python
from subprocess import check_output
print(check_output(["ls", "data"]).decode("utf8"))
```

    aisles.csv
    departments.csv
    order_products__prior.csv
    order_products__train.csv
    orders.csv
    products.csv
    sample_submission.csv
    Untitled Folder
    



```python
order_products_train_df = pd.read_csv("data/order_products__train.csv")
order_products_prior_df = pd.read_csv("data/order_products__prior.csv")
orders_df = pd.read_csv("data/orders.csv")
products_df = pd.read_csv("data/products.csv")
aisles_df = pd.read_csv("data/aisles.csv")
departments_df = pd.read_csv("data/departments.csv")
```
The first file includes a list of all orders, one row per order. For example, we can see that user 1 has 11 orders, 1 of which is in the train set, and 10 of which are prior orders. This file also contains the order number, the day of the week and hour of the day when the order was made and finally the days since the user’s prior order. The orders.csv doesn’t not include which products were purchased in each order. This is contained in the order_products.csv

```python
orders_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>order_id</th>
      <th>user_id</th>
      <th>eval_set</th>
      <th>order_number</th>
      <th>order_dow</th>
      <th>order_hour_of_day</th>
      <th>days_since_prior_order</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2539329</td>
      <td>1</td>
      <td>prior</td>
      <td>1</td>
      <td>2</td>
      <td>8</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2398795</td>
      <td>1</td>
      <td>prior</td>
      <td>2</td>
      <td>3</td>
      <td>7</td>
      <td>15.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>473747</td>
      <td>1</td>
      <td>prior</td>
      <td>3</td>
      <td>3</td>
      <td>12</td>
      <td>21.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2254736</td>
      <td>1</td>
      <td>prior</td>
      <td>4</td>
      <td>4</td>
      <td>7</td>
      <td>29.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>431534</td>
      <td>1</td>
      <td>prior</td>
      <td>5</td>
      <td>4</td>
      <td>15</td>
      <td>28.0</td>
    </tr>
  </tbody>
</table>
</div>


The second and third file specifies which products were purchased in each prior and train order accordingly. Order_products__prior.csv contains previous order contents for all customers, while Order_products__train.csv contains the last order of every user which is the order that we want to predict. The attribute ‘add_to_cart_order’ describes the order in which the user bought one specific product while completing his order. The attribute 'reordered' indicates that the customer has a previous order.that contains the product. 

```python
order_products_prior_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>order_id</th>
      <th>product_id</th>
      <th>add_to_cart_order</th>
      <th>reordered</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>33120</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>28985</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>9327</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2</td>
      <td>45918</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>30035</td>
      <td>5</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



```python
order_products_train_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>order_id</th>
      <th>product_id</th>
      <th>add_to_cart_order</th>
      <th>reordered</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>49302</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>11109</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>10246</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>49683</td>
      <td>4</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>43633</td>
      <td>5</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>

The fourth file contains the names of the products with their corresponding product_id. Furthermore the aisle and department are included.



```python
products_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>product_id</th>
      <th>product_name</th>
      <th>aisle_id</th>
      <th>department_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Chocolate Sandwich Cookies</td>
      <td>61</td>
      <td>19</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>All-Seasons Salt</td>
      <td>104</td>
      <td>13</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Robust Golden Unsweetened Oolong Tea</td>
      <td>94</td>
      <td>7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Smart Ones Classic Favorites Mini Rigatoni Wit...</td>
      <td>38</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Green Chile Anytime Sauce</td>
      <td>5</td>
      <td>13</td>
    </tr>
  </tbody>
</table>
</div>

The fifth file contains the different aisles in which the products belong.


```python
aisles_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>aisle_id</th>
      <th>aisle</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>prepared soups salads</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>specialty cheeses</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>energy granola bars</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>instant foods</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>marinades meat preparation</td>
    </tr>
  </tbody>
</table>
</div>

Finally, the fifth file contains the different departments in which the products belong.


```python
departments_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>department_id</th>
      <th>department</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>frozen</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>other</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>bakery</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>produce</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>alcohol</td>
    </tr>
  </tbody>
</table>
</div>


The dataset consists of information about 3.4 million grocery orders, distributed across 6 csv files as it was mentioned earlier. There are 206,209 customers in total. Out of which, the last purchase of 131,209 customers are given as train set and we need to predict for the rest 75,000 customers. The products belong to 134 aisles and the aisles belong to 21 departments.
