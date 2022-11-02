## Build LookML Objects in Looker: Challenge Lab

### Set up env parameters and download the necessary files :-

```yaml
export PROJECT=$(gcloud info --format='value(config.project)')

```


### Task - 1 : Create dimensions and measures :-

```yaml
dimension: is_search_source {
    type: yesno
    sql: ${users.traffic_source} = "Search" ;;
  }


  measure: sales_from_complete_search_users {
    type: sum
    sql: ${TABLE}.sale_price ;;
    filters: [is_search_source: "Yes", order_items.status: "Complete"]
  }


  measure: total_gross_margin {
    type: sum
    sql: ${TABLE}.sale_price - ${inventory_items.cost} ;;
  }


  dimension: return_days {
    type: number
    sql: DATE_DIFF(${order_items.delivered_date}, ${order_items.returned_date}, DAY);;
  }

```

### Task - 2 : Create a persistent derived table :-
### Task - 3 : Use Explore filters :-
Filter#1:  Add a filter to the Order Items Explore to include only items where the sale price is greater than or equal to 375. This will be used to omit any orders from the Explore that are less than $ 375
```yaml
always_filter:{
    filters:[order_items.sale_price: ">=375"]
}
```

Filter#2: Remove the previous filter. Next, add a filter to the Order Items Explore to only return data for orders that were shipped in the year 2018, unless a filter is applied to the Order Item Status or Order Item Delivered Date. Use the shipped_date dimension for this filter.

```yaml
conditonal_filter:{
    filters:[order_items.shipped_date: "2018"]
    unless: [order_items.status, order_items.delivered_date]
}
```

Filter#3: Remove the previous filter. Next, add a filter to the Order Items Explore to filter out all the items where the average sale price is more than 98 . That is, use a filter to only show orders with an average sale price of $ 98 or more.

```yaml
sql_always_having: ${average_sale_price} >=98
```

Filter#4: Remove the previous filter. Next, use a filter to define default values for the Order Status, State, and Traffic Source for the Order Items Explore. Make sure that the filter is required for the business user, but they will still be able to provide different values for these dimensions.

```yaml
always_filter:{
    filters:[order_items.status: "Shipped", user.state: "California", users.traffic_source:"Search"]
}
```
### Task - 4 : Apply a datagroup to an Explore :-
In training_ecommerce.model:  
1. Add datagroup

```yaml
datagroup: order_items_challenge_datagroup{
    sql_trigger: SELET MAX(order_item_id) from order_items ;;
    max_cache_age: "8 hour"

}
```

2. join 


```yaml
join: order_items_challenge{
    type: left_outer
    sql_on: ${order_items.order_id} = ${order_items_challenge.order_id}
    relationship: many_to_one
}
```

Commit Changes and Push > Deploy to Production

Congratulations! :beer: :beer: