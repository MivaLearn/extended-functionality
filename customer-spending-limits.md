## Calculate Customer Order Total
- Create customer custom field
- Create template feed
    - Pull in all store customers
        - Use [CustomerList_Load_All](https://docs.miva.com/api-functions/customerlist_load_all) to pull in all customers
        - Pull in all orders by customers
            - Use [OrderList_Load_Customer](https://docs.miva.com/api-functions/orderlist_load_customer) to pull in customer orders.
            - Calculate captured total
            - Add captured total to customer custom field
- Create scheduled task to run feed
<br><br><br>

## Update Customer Order Total
- Create ReadyTheme Content Section
    - Pull in customer custom field
        ```xml
        <mvt:item name="customfields" param="Read_Customer_ID( CUST_ID, 'custom_field_code', 'value' )" />
        ``` 
    - Create logic to update customer custom field 
        ```xml
        <mvt:item name="customfields" param="Write_Customer_ID( CUST_ID, 'custom_field_code', 'value' )" />
        ```     
    - Show message that display's remaining balance
- Add content section to INVC (using a content section makes it easier to maintain the code for future updates)
<br><br><br>

## Checkout Conditional Logic
- Create ReadyTheme Content Section
    - Pull in customer custom field
        ```xml
        <mvt:item name="customfields" param="Read_Customer_ID( CUST_ID, 'custom_field_code', 'value' )" />
        ``` 
    - Write logic to check if customer meets requirement
    - If customer does not meet requirement redirect them to the BASK page
        - Add [302 Redirect](https://snippets.cacher.io/snippet/aa6d1e0bb473ad91579a) to BASK page with error parameter passed
        - Add error message on BASK page to inform customer of spending limit
- Add Content section to top of BASK, OCST, OSEL, OPAY


        