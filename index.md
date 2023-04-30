---
title: Instacart Market Basket Analysis
layout: minimal
---

<style>
  .box { display: inline-block; }
  /* .left {align: left; }
  .right {align: right; } */
</style>

# 🛒 Instacart Market Basket Analysis 
{:.no_toc}


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Summary and Business Recommendations

<ol>
  <li>Order volumes are considerably higher during daytime hours on weekends (9 AM to 5 PM). Therefore, it is recommended that the company allocates more drivers during these peak hours by adjusting delivery fees. This would ensure sufficient driver availability and timely order deliveries, leading to increased customer satisfaction and loyalty. </li>
  <li>The majority of orders (~48%) are placed more than 7 days after the customer's last order. To encourage more frequent purchase, it is suggested that the company utilizes targeted strategies to increase customer loyalty and drive repeat business. For example, coupons, discounts and free shipping can be offered to customers whose second order exceeds a certain amount within a week. </li>
  <li>Among the top 20 products, 14 (~75%) are organic. Organic products also have higher reorder rates compared to non-organic products. Given the growing demand for healthier food options, the company should consider increasing the visibility of organic products across its websites and mobile applications. </li>
  <li>The company may consider building a dynamic dashboard based on the treemap (<a href="#treemap">Figure 10</a>) with regular updates. It can facilitate ongoing monitoring of overall sales conditions, streamline data storage and analysis, and improve decision making efficiency. </li>
</ol>


## Data Preprocessing
The original datasets were obtained from Kaggle. There are five original datasets, which respectively contain information on order information, products purchased in each order, product information, department information and aisle information from Instacart, a grocery delivery service. 
<br/><br/>
During dataset joining, there seems to be a problem of mismatched numbers of rows between datasets, which may indicate many-to-one or duplicate issues. After investigating the unique values in datasets that contain department and aisle information, two departments and three aisles with repetitive ids and unreasonable names were dropped. It is more common to reach out to data providers when dealing with this kind of issue in practice, but here intuition works just fine since stores probably do not sell “illegal drugs” or “nuclear missiles”.
<br/><br/>
There are also some messy entries of aisle names. Necessary actions have been done to clean up these data.
<table style="width:70%;margin:auto;">
  <tr>
    <th>Original</th>
    <th>Cleaned</th>
  </tr>
  <tr>
    <td>1_!ice68'_cream68'_toppings</td>
    <td>ice cream toppings</td>
  </tr>
  <tr>
    <td>10_!bakery68'_desserts</td>
    <td>bakery desserts</td>
  </tr>
</table>
<p style="font-size:85%;text-align:center;">Table 1: Example of Data Cleaning Process</p>


## Exploratory Data Analysis 
### Customer Behavior 

<iframe src="assets/num_orders_per_user_1" width="100%" height=530  frameBorder=0></iframe>
<p style="font-size:85%;text-align:center;">Figure 1: Distribution of Number of Orders Per User</p>

There are approximately 200k users, 3.2 million orders and 50k unique product offerings according to the dataset. As is demonstrated in Figure 1, the number of orders per user has a negative exponential distribution. Around 29% of users placed orders 1-5 times, about 25% of users placed orders 6-10 times, and no more than 9% of users had ordered more than 40 times. There is a peak at 99 orders per user, which may indicate potential data issues.

<iframe src="assets/dow_tod_heatmap_2" width="100%" height=530  frameBorder=0></iframe>
<p style="font-size:85%;text-align:center;">Figure 2-1: Heatmap of Order Volume by Day of Week and Time of Day</p>

According to Figure 2-1, more than 34% of the orders are placed on Saturday and Sunday, and about 72% of orders were placed during daytime hours (i.e. 9 AM to 5 PM), which aligns with normal working hours.

**Therefore, it is recommended that the company increase the delivery fee during peak hours, specifically from 9 AM to 5 PM on weekends, to ensure an adequate number of available drivers. Such an approach would guarantee timely order deliveries and  increase customer satisfaction and loyalty.**

<div class="box">
  <iframe align="left" src="assets/dow_bar_11" height=500 frameBorder=0></iframe>
  <iframe align="right" src="assets/tod_bar_12" height=500 frameBorder=0></iframe>
</div>

<!-- <div class="box">
<p style="font-size:85%;text-align:center;">Figure 2-2: Distribution of Orders by Day of Week</p>
<p style="font-size:85%;text-align:center;">Figure 2-3: Distribution of Orders by Time of Day</p>
</div> -->

Note: The day of week in the original dataset is marked with mere numbers from 0 to 6. This analysis assumes that day 0 is Saturday and day 1 is Sunday given that people tend to shop for groceries more on weekends. Another point to note is that day 2 has the most orders among the rest of the days. Such a phenomenon supports the above assumption of days of weeks because intuitively people are likely to shop for what they’ve missed in weekend shopping on Monday. More details are shown in figure 2-2 and 2-3. <u>A follow up with data providers would still be helpful.</u>

<br/><br/>

<iframe src="assets/order_size_hist_3" width="100%" height=530  frameBorder=0></iframe>
<p style="font-size:85%;text-align:center;">Figure 3: Distribution of Order size</p>

As for the number of products per order, Figure 3 indicates that over 62% of orders contain no more than 10 products, and less than 10% of orders have more than 20 items.

<br/><br/>

<iframe src="assets/day_since_prior_order_4" width="100%" height=530  frameBorder=0></iframe>
<p style="font-size:85%;text-align:center;">Figure 4: Distribution of Day Since Prior Order</p>

When looking closely at Figure 4, the distribution of the number of days customers placed their prior order, there seems to be a weekly user cycle since there’s a peak every seven days. <u>There is a peak on day 30, which probably implies that the data maintainer aggregates any data larger than 30 into that bin. A follow up investigation is needed.</u>

<iframe src="assets/second_order_size_5" width="100%" height=530  frameBorder=0></iframe>
<p style="font-size:85%;">Figure 5: Probability Density Histogram of Second Order Size relative to individual subpopulations. Orders are divided into three subpopulations by time since last order: “Same day” (<code>day_since_prior_order</code> = 0), “Within a week” (0 < <code>day_since_prior_order</code> <= 7), and “More than a week” (<code>day_since_prior_order</code> > 7). Distribution of order size of the subpopulations can be found in the <a href="#appendix">appendix</a>.

When comparing the distribution of order size across subpopulations with varying time intervals since the prior order, as shown in Figure 5, there seem to be more small-sized second orders placed on the same day, which indicates that people who reordered on the same day may just forget a thing or two.

Looking closer at the individual subpopulations, it's worth noting that only about 2% of orders fall into the "same day" category, while no more than 40% of orders are placed within a week of the customer's previous order. The majority of orders (~48%) fall into the "more than a week" category. Additionally, the median order sizes for the second order in the "same day", "within a week", and "more than a week" subpopulations are 5, 8, and 9, respectively. This aligns with the common understanding that people tend to purchase more items when placing an order after a longer interval of time.

**As a result, to encourage customers who place orders more than a week apart to shift into the "within a week" category, it is recommended that marketing, sales, and other relevant departments collaborate to develop targeted strategies. One approach could be to offer coupons, discounts, or free shipping for customers whose second order exceeds a certain amount within a week.**

</p>

### Product
As for individual products, the most ordered product is banana. The top 20 products with highest volume percentages are all from the `produce` and `dairy eggs` departments. Produce and vegetables expire much quicker than the other product offerings and are more commonly in everyday dishes/meals which aligns with what might be expected. Among the top 20 products, 14 (~75%) are organic. Furthermore, organic products have higher reorder rates compared to non-organic products.

**With the increasing trend towards natural and healthy foods, this presents a promising opportunity for the company to explore further. For instance, the company could enhance its promotion of organic products by increasing their visibility on its websites and apps.**

<table style="width:70%;margin:auto;">
  <tr>
    <th style="text-align: center">Product Name</th>
    <th style="text-align: center">Volume Percentages</th>
    <th style="text-align: center">Reorder Rate</th>
  </tr>
  <tr>
  <td>Banana</td>
  <td style="text-align: right">    1.46%</td>
  <td style="text-align: right">    0.8435</td>
  </tr>
  <tr>
  <td>Bag of Organic Bananas</td>
  <td style="text-align: right">    1.17%</td>
  <td style="text-align: right">    0.8326</td>
  </tr>
  <tr>
  <td>Organic Strawberries</td>
  <td style="text-align: right">    0.82%</td>
  <td style="text-align: right">    0.7777</td>
  </tr>
  <tr>
  <td>Organic Baby Spinach</td>
  <td style="text-align: right">    0.75%</td>
  <td style="text-align: right">    0.7725</td>
  </tr>
  <tr>
  <td>Organic Hass Avocado</td>
  <td style="text-align: right">    0.66%</td>
  <td style="text-align: right">    0.7966</td>
  </tr>
  <tr>
  <td>Organic Avocado</td>
  <td style="text-align: right">    0.54%</td>
  <td style="text-align: right">    0.7581</td>
  </tr>
  <tr>
  <td>Large Lemon</td>
  <td style="text-align: right">    0.47%</td>
  <td style="text-align: right">    0.696</td>
  </tr>
  <tr>
  <td>Strawberries</td>
  <td style="text-align: right">    0.44%</td>
  <td style="text-align: right">    0.6982</td>
  </tr>
  <tr>
  <td>Limes</td>
  <td style="text-align: right">    0.43%</td>
  <td style="text-align: right">    0.681</td>
  </tr>
  <tr>
  <td>Organic Whole Milk</td>
  <td style="text-align: right">    0.43%</td>
  <td style="text-align: right">    0.8304</td>
  </tr>
  <tr>
  <td>Organic Raspberries</td>
  <td style="text-align: right">    0.42%</td>
  <td style="text-align: right">    0.7691</td>
  </tr>
  <tr>
  <td>Organic Yellow Onion</td>
  <td style="text-align: right">    0.35%</td>
  <td style="text-align: right">    0.6971</td>
  </tr>
  <tr>
  <td>Organic Garlic</td>
  <td style="text-align: right">    0.33%</td>
  <td style="text-align: right">    0.6801</td>
  </tr>
  <tr>
  <td>Organic Zucchini</td>
  <td style="text-align: right">    0.32%</td>
  <td style="text-align: right">    0.6884</td>
  </tr>
  <tr>
  <td>Organic Blueberries</td>
  <td style="text-align: right">    0.31%</td>
  <td style="text-align: right">    0.6288</td>
  </tr>
  <tr>
  <td>Cucumber Kirby</td>
  <td style="text-align: right">    0.3%</td>
  <td style="text-align: right">    0.6917</td>
  </tr>
  <tr>
  <td>Organic Fuji Apple</td>
  <td style="text-align: right">    0.27%</td>
  <td style="text-align: right">    0.7119</td>
  </tr>
  <tr>
  <td>Organic Lemon</td>
  <td style="text-align: right">    0.27%</td>
  <td style="text-align: right">    0.6899</td>
  </tr>
  <tr>
  <td>Apple Honeycrisp Organic</td>
  <td style="text-align: right">    0.26%</td>
  <td style="text-align: right">    0.7352</td>
  </tr>
  <tr>
  <td>Organic Grape Tomatoes</td>
  <td style="text-align: right">    0.26%</td>
  <td style="text-align: right">    0.6555</td>
  </tr>
</table>

<p style="font-size:85%;text-align:center;">Table 2: Top 20 Best Selling Products With Reorder Rates</p>

In Figure 6 and Figure 7, the most reordered ones are mostly food, drinks, and personal care items, which are the ones that get used up pretty quickly.

<iframe src="assets/dept_reorder_6" width="100%" height=530  frameBorder=0></iframe>
<p style="font-size:85%;text-align:center;">Figure 6: Department Reorder Rates</p>

<iframe src="assets/aisle_reorder_7" width="100%" height=530  frameBorder=0></iframe>
<p style="font-size:85%;text-align:center;">Figure 7: Aisle Reorder Rates</p>

According to Figure 8, the department with the largest order volume is `produce`. The `other` category in the pie chart is made up of departments with less than 2% of total order volumes, which includes `personal care`, `babies`, `international`, `alcohol`, `pets`, `missing`, `other` and `bulk`. 

<iframe src="assets/order_dept_pie_8" width="100%" height=530  frameBorder=0></iframe>
<p style="font-size:85%;text-align:center;">Figure 8: Pie Chart of Order Volume Percentage By Department</p>

According to Figure 9, the single aisle with the largest order volume is `fresh fruits`. The aisles that are not among the top 15 ones with most order volumes are compiled into the `other` category due to space limitations. 

<iframe src="assets/order_aisle_pie_9" width="100%" height=530  frameBorder=0></iframe>
<p style="font-size:85%;text-align:center;">Figure 9: Pie Chart of Order Volume Percentage By Aisle</p>

The treemap below (Figure 10) provides a visual representation of order volumes and reorder rates across various departments and aisles, using size and color to convey information. **To facilitate ongoing monitoring of overall sales conditions, the company may consider transforming this treemap into a dynamic dashboard that can be accessed by product managers and other senior staff. Implementing this approach can also help optimize data storage and analysis as well as enhance the efficiency of decision-making process.**

<iframe src="assets/treemap_10" id="treemap" width="100%" height=530  frameBorder=0></iframe>
<p style="font-size:90%;">Figure 10: Treemap of Department and Aisles, sized by order volumes and colored by reorder rates. More detailed data can be found by hovering over the cells.</p>


## Future work
The questions that require follow up with the data providers include the following:

Improvement to data quality:
<ul>
  <li>Which specific days of week do the numbers in the dataset correspond to?</li>
  <li>Under what mechanism is a product classified into the missing department or aisle?</li>
  <li>Regarding the unusual peak on day 30 in the distribution of days since prior order, is any data larger than 30 recorded as 30?</li>
</ul>

Extra information:
<ul>
  <li>Can other data on revenue and time series be provided?</li>
</ul>

With these questions answered, it is possible to further enhance the insights provided above.

<!-- ## Appendix -->


<!-- <iframe src="assets/size_sub1_13" id="treemap" width="100%" height=530  frameBorder=0></iframe>
<p style="font-size:85%;text-align:center;">Figure 13: Size of Second Order - Same day</p>

<iframe src="assets/size_sub2_14" id="treemap" width="100%" height=530  frameBorder=0></iframe>
<p style="font-size:85%;text-align:center;">Figure 14: Size of Second Order - Within a week</p>

<iframe src="assets/size_sub3_15" id="treemap" width="100%" height=530  frameBorder=0></iframe>
<p style="font-size:85%;text-align:center;">Figure 15: Size of Second Order - More than a week</p> -->