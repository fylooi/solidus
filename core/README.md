Summary
------
Solidus Core provides the essential e-commerce data models upon which the
Solidus system depends.

Core Models
-----------
Solidus implements over 200 [models](https://github.com/solidusio/solidus/tree/master/core/app/models/spree),
and thus a deep inspection of each in this README would be overkill. Instead,
let's take a quick look at the fundamental models upon which all else depend.
Currently, these models remain in the Spree namespace as part of the legacy of
[forking Spree](https://solidus.io/blog/2015/10/28/future-of-spree.html).

## NOTE: Documentation is a work in progress
The documentation of Solidus Core is still in progress. Contributions following
this form are welcome and encouraged!

* [The Product Sub-System](#the-product-sub-system)
* [The Promotion Sub-System](#the-promotion-sub-system)
* [The Order Sub-System](#the-order-sub-system)
* [The Payment Sub-System](#the-payment-sub-system)
* [The Return Sub-System](#the-return-sub-system)
* [The User Sub-System](#the-user-sub-system)
* [The Inventory Sub-System](#the-inventory-sub-system)
* [The Shipments Sub-System](#the-shipments-sub-system)

## The Product Sub-System
Products represent an entity for sale in a store. Each product may have 
multiple variants with different options - eg. Size (S / M / L). Products 
support user defined properties and may be categorized into multiple groupings

* `Spree::Product` - Records attributes that do not change by variant 
(description, permalink, availability, shipping category, etc)
* `Spree::Variant` - Individual product variations and the attributes that may 
vary, eg. price, sku 
* `Spree::OptionType` &  `Spree::OptionValue` - User defined product 
variation attributes (eg. Size - S/M/L)
* `Spree::Property` - User defined attributes for products, primarily used for 
presentational purposes (eg. Material - Velvet)
* `Spree::Classification` - Categorizes the product as belonging to a particular 
group (`Spree::Taxon`). Products may have multiple classifications. 
(eg. clothes, top, skirt)

## The Promotion Sub-System
Promotions represent a set of rules which apply discounts to a customer's 
order. These discounts may be applied by coupon, in cart, towards shipping and 
by visiting a specific page.

* `Spree::Promotion` - Stores promotion attributes and applies to orders
* `Spree::PromotionRule` - Business logic to determine whether a promotion is 
applicable to an order
* `Spree::PromotionAction` - Applies a promotion's adjustment (total, 
line item total, item quantity, free shipping) to the order
* `Spree::PromotionCode` - Tracks discount codes and usage restrictions
* `Spree::PromotionHandler::*` - Activates applicable promotions based on 
context (eg. coupon, cart, page, shipping)

## The Order Sub-System
An order keeps track of the customer's cart until check out is completed, 
then acts as permanent record of the transaction. Orders contain line items.  
Order prices may be modified (by tax rates, promotions, shipping, etc.).  
These price changes are recorded as adjustments.

* `Spree::Order` - the heart of the Solidus system, as it acts as the customer's
cart as they shop. Once an order is complete, it serves as the permanent 
record of their purchase. 
* `Spree::Store` - Records store specific configuration such as store name and URL.
permenent record of the transaction.
* `Spree::LineItem` - Variants placed in the order at a particular price.
* `Spree::Adjustment` - Represents a change to the item total of an order

## The Payment Sub-System
* `Spree::Payment` - Manage and process a payment for an order, from a specific
source (e.g. `Spree::CreditCard`) using a specific payment method (e.g
`Solidus::Gateway::Braintree`).
* `Spree::PaymentMethod` - An abstract class which is implemented most commonly
as a `Spree::Gateway`.
* `Spree::Gateway` - A concrete implementation of `Spree::PaymentMethod`
intended to provide a base for extension. See
https://github.com/solidusio/solidus_gateway/ for offically supported payment
gateway implementations.
* `Spree::CreditCard` - The default `source` of a `Spree::Payment`.

## The Return Sub-System
* `Spree::CustomerReturn` - Logs a set of returned items at a stock location. 
* `Spree::ReturnItem` - Tracks the return status of individual returned items 
(eg reimbursed, exchanged, etc)
* `Spree::Reimbursement` - Manages reimbursements which are in the form of 
store credit, refunds or exchanges
* `Spree::StoreCredit` - Tracks and controls usage of the customer's store 
credit
* `Spree::Refund` - Performs refunds for a given payment 

## The User Sub-System
* `Spree::LegacyUser` - Default implementation of User.
* `Spree::UserClassHandle` - Configuration point for User model implementation.
* [solidus_auth_devise](https://github.com/solidusio/solidus_auth_devise) -
An offical, more robust implementation of a User class with Devise
integration.

## The Inventory Sub-System
Spree allows for inventory tracking in multiple locations. Every variant 
stocked at a particular location is tracked as an individual stock item with 
quantity on hand. Inventory quantity between locations may be moved using 
stock movements. Stock location may have location specific shipping methods.

Every line item for a given order is split into individual inventory units. 
Each inventory unit denotes one unit of the product variant and is 
individually tracked for fulfillment. 

* `Spree::ReturnAuthorization` - Models the return of Inventory Units to
a Stock Location for an Order.
* `Spree::StockLocation` - Records the name and addresses from which stock items
are fulfilled in cartons.
* `Spree::StockItem` - Tracks quantity at hand for product variants in stock. 
Also handles back orders
* `Spree::InventoryUnit` - Tracks the state of line items' fulfillment.

## The Shipments Sub-System
The Shipments subsystem is designed for customization. Shipping methods may be 
constrained by geographic zone or product(s). Shipping rate calculation is 
implemented using calculators.

* `Spree::Shipment` - An order's planned shipments including
tracking and cost. Shipments are fulfilled from Stock Locations.
* `Spree::ShippingMethod` - Represents a means of having a shipment delivered,
such as FedEx or UPS.
* `Spree::Zone`, `Spree::ShippingMethodZone` - Represents a group of zone 
members (countries and / or states) and the shipping methods available to 
those zones. 
* `Spree::ShippingCategory` - User created categories for shipping methods. 
Used to implement shipping method restrictions based on products (eg. oversized)
* `Spree::ShippingRate` - Records the costs of different shipping methods for 
a shipment and which method has been selected to deliver the shipment.
* `Spree::Calculator` - Base class used to implement different types of 
calculations. Calculators currently implemented for shipping are: 
  - Flat percent (eg. 10% of item total)
  - Flat rate (eg. $5)
  - Flexible rate (eg. $10 for first item, $5 for next 3 items, $0 beyond that)
  - Per item (eg. $1 per item)
  - Price sack (eg. $10 discount on shipping for orders over $50)
* `Spree::Carton` - Used to group shipment inventory units into shipping 
cartons for logistics tracking 

Developer Notes
---------------
## Testing

Create the test site

    bundle exec rake test_app

Run the tests

    bundle exec rake spec
