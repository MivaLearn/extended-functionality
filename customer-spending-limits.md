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
- Add content section to ```INVC``` (using a content section makes it easier to maintain the code for future updates)
<br><br><br>

## Checkout Conditional Logic
- Create ReadyTheme Content Section
    - Pull in customer custom field
        ```xml
        <mvt:item name="customfields" param="Read_Customer_ID( CUST_ID, 'custom_field_code', 'value' )" />
        ``` 
    - Write logic to check if customer meets requirement
    - If customer does not meet requirement redirect them to the ```BASK``` page
        - Add 302 Redirect to ```BASK``` page with error parameter passed
        ```xml 
        <mvt:comment> Get Customer custom field that holds the customer YTD order total </mvt:comment>
        <mvt:item name="customfields" param="Read_Customer_ID( g.customer:id, 'custom_field_code', l.settings:customer:order_tot )" />
        <mvt:comment> Add current basket total to customer's order total </mvt:comment>
        <mvt:assign name="l.settings:customer:future_total" value="l.settings:global_minibasket:total + l.settings:customer:order_tot" />
        <mvt:comment> read a customer customfield that holds the customer spending limit </mvt:comment>
        <mvt:item name="customfields" param="Read_Customer_ID( g.customer:id, 'custom_field_code', l.settings:customer:spending_limit )" />
        <mvt:if expr="l.settings:customer:future_total GT l.settings:customer:spending_limit">
            <mvt:if expr="l.settings:page EQ 'BASK'">
                <mvt:comment> Set variable to show error message </mvt:comment>
                <mvt:assign name="g.spending_limit_error" value="'exceeded-limit'" />
            <mvt:else>
                <mvt:do file="g.Module_Feature_TUI_DB" name="l.success" value="Page_Load_Code('BASK', l.settings:basket_page)" />
                <mvt:assign name="l.uri:store_id" value="g.Store:id" />
                <mvt:assign name="l.uri:screen" value="''" />
                <mvt:assign name="l.uri:page_id" value="l.settings:basket_page:id" />
                <mvt:assign name="l.uri:cat_id" value="0" />
                <mvt:assign name="l.uri:product_id" value="0" />
                <mvt:comment>Load canonical URI for BASK</mvt:comment>
                <mvt:do file="g.Module_Feature_URI_DB" name="l.has_uri" value="URI_Load_Item_Canonical( l.uri, l.settings:basket:uri )" />
                <mvt:if expr="l.has_uri">
                    <mvt:comment>Add error parameter to redirect URL</mvt:comment>
                    <mvt:assign name="l.settings:basket:uri:redirect_to" value="l.settings:basket:uri:uri $ '?spending_limit_error=exceeded-limit'" />
                    <mvt:assign name="l.header" value="miva_output_header( 'Status', '302 Found' )" />
                    <mvt:assign name="l.header" value="miva_output_header( 'Location', l.settings:basket:uri:redirect_to  )" />
                </mvt:if>
            </mvt:if>
        </mvt:if>
        ```
        - Add error message on BASK if ```g.spending_limit``` is equal to ```exceeded-limit``` to inform customer of spending limit error
- Add Content section to top of ```BASK```, ```OCST```, ```OSEL```, ```OPAY```
