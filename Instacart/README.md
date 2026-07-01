# Instacart EDA

This is the 2nd project I worked on in the TripleTen Data Science program. 

### Objective

To clean up the data and prepare a report that gives insight into the shopping habits of Instacart customers.

### Data Dictionary

There are five tables in the dataset. Below is a data dictionary that lists the columns in each table and describes the data they hold.

- `instacart_orders.csv`: each row corresponds to one order on the Instacart app
    - `'order_id'`: ID number that uniquely identifies each order
    - `'user_id'`: ID number that uniquely identifies each customer account
    - `'order_number'`: the number of times this customer has placed an order
    - `'order_dow'`: day of the week that the order placed (which day is 0 is uncertain)
    - `'order_hour_of_day'`: hour of the day that the order was placed
    - `'days_since_prior_order'`: number of days since this customer placed their previous order
- `products.csv`: each row corresponds to a unique product that customers can buy
    - `'product_id'`: ID number that uniquely identifies each product
    - `'product_name'`: name of the product
    - `'aisle_id'`: ID number that uniquely identifies each grocery aisle category
    - `'department_id'`: ID number that uniquely identifies each grocery department category
- `order_products.csv`: each row corresponds to one item placed in an order
    - `'order_id'`: ID number that uniquely identifies each order
    - `'product_id'`: ID number that uniquely identifies each product
    - `'add_to_cart_order'`: the sequential order in which each item was placed in the cart
    - `'reordered'`: 0 if the customer has never ordered this product before, 1 if they have
- `aisles.csv`
    - `'aisle_id'`: ID number that uniquely identifies each grocery aisle category
    - `'aisle'`: name of the aisle
- `departments.csv`
    - `'department_id'`: ID number that uniquely identifies each grocery department category
    - `'department'`: name of the department

The data is provided by TripleTen, modified from Instacart via Kaggle.

### Process

First, I opened the data files and reviewed  the general contents of each table. 

Next, I preprocessed the data by:
* Verifying and fixing data types (e.g. make sure ID columns are integers)
* Identifying and filling in missing values
* Identifying and removing duplicate values

I then performed deeper exploratory data analysis (EDA), including:
1. Verifying that values in the 'order_hour_of_day' and 'order_dow' columns in the orders table are sensible.
2. Creating a plot that shows how many people place orders for each hour of the day.
3. Creating a plot that shows what day of the week people shop for groceries.
4. Creating a plot that shows how long people wait until placing their next order, and comment on the minimum and maximum values.
5. Exporing if there is a difference in 'order_hour_of_day' distributions on Wednesdays and Saturdays.
6. Plotting the distribution for the number of orders that customers place.
7. Identifying the top 20 products that are ordered most frequently.
8. Identifying how many items people typically buy in one order.
9. Identifying the top 20 items that are reordered most frequently.
10. Identifying the proportion of each product’s orders that are reorders.
11. Idenfiying the  proportion of products in each cutomer’s orders that are reorders.

### Results

This exploratory data analysis (EDA) on Instacart transactions yields a number of preliminary insights that can inform further analysis.

After identifying patterns in customer habits, it could be worth exploring any correlations between the observed patterns and existing app features. For example, are the peak ordering times of 10 AM and 3 PM correlated with the timing of app push notifications? Do the top ordered items correlate with the most suggested items in the app? <br><br> Depending on which features currently exist in the Instacart app, the EDA might also provide some insights to guide exploration for possible adjustments or even new features. For example, are push notifications currently tuned to reflect differences in peak ordering hours over different days of the week? Are product suggestions for loyal customers tuned to their individual order preferences? And, considering the large share of customers that drop off after placing one order, perhaps offering a coupon code could entice them to come back. <br><br>Overall, the patterns revealed in this analysis make intuitive sense and provide a strong foundation to dig deeper.

Please see the included Jupyter Notebook for the full analysis.
