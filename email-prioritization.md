#WooCommerce Email Queuing Priorities

## 1. Product Matches

Exact product matches are processed first, followed by always_send product matches.

- Look for specific product matches that are <code>always_send=off</code> and <code>email_type!=storewide</code>
- Skip date emails that have send dates in the past
- Store remaining matches sorted by <code>priority</code> as <code>$emails</code>

## 2. "Always Send" Product Matches

<code>always_send</code> product matches run through the same process as above but with <code>always_send</code> property set to <code>on</code> and that matches are stored as <code>$always_prods</code>.

## 3. "Always Send" Category Matches

All product categories in the given order are pulled and stored in <code>$all_categories</code>. <code>always_send</code> category emails are loaded first before the category emails with <code>always_send=off</code>

- Search for emails that matches any of the category in <code>$all_categories</code>
- Skip date emails that have send dates in the past
- Store remaining matches sorted by <code>priority</code> as <code>$always_cats</code>

## 4. Process "Always Send" Product Matches

Before adding all <code>$always_prods</code> emails to the queue, the code first checks for and queues <code>reminder</code> emails.

After processing and queuing any <code>reminder</code> emails, the emails inside <code>$always_prods</code> are then inserted into the queue.

## 5. Process "Always Send" Category Matches

This is a straight-forward process of looping through the <code>$always_cats</code> array and adding items to the email queue.

## 6. Process Exact Product Matches

- Get the email with the highest priority and insert it into the email queue
- Loop through the other emails and add to queue if the product IDs are the same with the top email
- For the remaining emails, add them to the queue if the scheduled send date is within 60 minutes. Otherwise, ignore the email

## 7. Process Category Matches

- Get the email with the highest priority and insert it into the email queue
- Loop through the other emails and add to queue if the category IDs are the same with the top email
- For the remaining emails, add them to the queue if the scheduled send date is within 60 minutes. Otherwise, ignore the email

## 8. Storewide (Generic) Emails

A Storewide email will only be processed if no other emails have been queued for a given order. An exception to this rule is if the <code>always_send</code> property is set to <code>on</code>.

Other plugins or add-ons (e.g. Subscriptions) can prevent FUE from sending any storewide emails for a particular order by hooking to the <code>fue_order_created</code> action and returning a boolean <code>true</code>.

- Load all storewide emails
- Skip emails that have <code>exclude_categories</code> set when an excluded category matches a category in <code>$all_categories</code>
- Also skip date emails that have send dates in the past
- Insert the remaining storewide emails to the email queue

## 9. First Purchase & Product Purchase Above One

#### Product Match

- Loop through the order items and get the number of times the current customer have purchased item
- If it is the first time, load and queue all <code>first_purchase</code> emails that matches the current order item ID
- If it is not the first time, load and queue all <code>product_purchase_above_one</code> emails that matches the current order item ID

#### Category Match

- Loop through the categories and get the number of times the current customer have purchased items under the categories
- If it is the first time, load and queue all <code>first_purchase</code> emails that matches the current category ID
- If it is not the first time, load and queue all <code>product_purchase_above_one</code> emails that matches the current category ID

## 10. Storewide First Purchase

- If the customer is a first-time purchaser, load all <code>first_purchase</code> emails that are <code>generic</code> and add them to the queue

## 11. Customer Emails

Load all <code>customer</code> emails and depending on the email triggers, add to the queue all emails that matches the email's filter (e.g. total purchases > 3).

## 12. Customer Emails - Last Purchased

Load all <code>customer</code> email with the trigger <code>after_last_purchase</code> and automatically add them all to the email queue. 

