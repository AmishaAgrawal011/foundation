# SQL Assignment Bank

These are SQL prompts that can be used as assingmnets or evaluation questions for someone learning SQL. Answering these will require a deep understanding of the OFBiz datamodel (UDM?) as well as how business processes connect to that model because every prompt will not specify which tables need to be quiered.

**Non Return Refunds**

Get a list of all refund payments made on orders due to cancelations. No return refunds should be included.

The expected fields in the result are:
1. Order Id
2. Payment internal Id
3. Payment manual reference
4. Payment parent reference
5. Refund amount from payment pref

**Digital Order Items**

Fetch a list of completed order items that were not POS Completed sales but also did not need to be shipped to the customer.

Fields to include in results:
1. Order Id
2. Item Seq Id
3. Item Attr (select any)
4. Item total

**Duplicate Orders**

Fetch a list of all orders imported from Shopify that are duplicates. There can be more than 1 duplicate.

Fields to include in results:
1. Order Name
2. Order Id
3. Duplicate of Order Id

**Kit Return Inventory Update**

A return was received and completed for a kit product (marketing package). Write an SQL to calculate how much QoH was increased for its copmonents.

Fields:
1. Return ID
2. Return Item Seq Id
3. Return Item Product Id
4. Marketing Package Component Id
5. Marketing Package Component QoH Increase

**Shipping Distance**

Calculate the average distance transfer orders are being shipped when moving inventory between two facilities. Facilities should have a latitude and longitude configured.

Fields:
1. Transfer Orer Name
2. Transfer Order Origin Facility Id
3. Transfer Order Destination Facility Id
4. Distance in Kilometers
5. Tracking Code
6. Label Cost

**Net Transfers**

At the end of every month, a retailer wants to see effectivley how much inventory was moved between facilitieis. Compute the net amount of transfers between all facilities. If two facilities transfered a product back and forth with quantity 3 and 2 respectively, the net transfer is 1.

Fields:
1. Facility 1
2. Facility 2
3. Product Internal Name
4. Net Transfered Qty

**Variances by External System**

Find a list of all inventory variances created by external systems. The variance reason used by the external system is "VAR_EXT_RESET" or something similar.

Fields:
1. Facility Id
2. Product Internal Name
3. Date Time
4. Variance Qty

**Order Import Taking Too Long**

There is a suspicion that orders are taking too long to be imported from Shopify. Trace how long each step of the order import is taking to find where lag is being introduced.