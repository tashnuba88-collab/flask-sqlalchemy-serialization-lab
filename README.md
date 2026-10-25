# Flask-SQLAlchemy Serialization Lab

A small e-commerce backend built with Flask, SQLAlchemy, and Marshmallow. It stores customers, the items they buy, and the reviews they leave, and it turns that data into nested dictionaries that are safe to send as JSON.

## Features

- Three models: `Customer`, `Item`, and `Review`.
- `Review` is a join table. A review belongs to one customer and one item, so customers and items are linked many-to-many through reviews.
- `Customer.items` is an association proxy that returns the items a customer has reviewed, without looping through reviews by hand.
- Marshmallow schemas (`CustomerSchema`, `ItemSchema`, `ReviewSchema`) that nest related records with `fields.Nested`.
- `exclude` rules on every nested field, which stop the infinite recursion you get when a customer contains reviews that contain the same customer again.

## How the models connect

```
Customer 1 --- * Review * --- 1 Item
```

- `Customer.reviews` and `Review.customer` use `back_populates`.
- `Item.reviews` and `Review.item` use `back_populates`.
- `Customer.items` goes through `reviews` to `item`.

## Serialization rules

Each schema serializes every column, including `id`, but leaves out the foreign keys (`customer_id`, `item_id`).

| Schema | Nested field | Excluded inside the nested field |
| --- | --- | --- |
| `CustomerSchema` | `reviews` | `customer` |
| `ItemSchema` | `reviews` | `item` |
| `ReviewSchema` | `customer` | `reviews` |
| `ReviewSchema` | `item` | `reviews` |

## Setup

1. Fork and clone this repo.
2. Install dependencies and open the virtual environment:

   ```console
   $ pipenv install
   $ pipenv shell
   ```

3. Move into the `server` directory:

   ```console
   $ cd server
   ```

4. Create the database tables:

   ```console
   $ flask db init
   $ flask db migrate -m "initial migration"
   $ flask db upgrade head
   ```

5. Add the `Review` table:

   ```console
   $ flask db migrate -m "add review"
   $ flask db upgrade head
   ```

6. Load the sample data:

   ```console
   $ python seed.py
   ```

## Trying it out

Open a Flask shell and look at a customer's reviewed items:

```py
>>> from models import *
>>> customer1 = Customer.query.filter_by(id=1).first()
>>> customer1.items
[<Item 1, Laptop Backpack, 49.99>, <Item 2, Insulated Coffee Mug, 9.99>]
>>> CustomerSchema().dump(customer1)
```

## Running the tests

From the `server` directory:

```console
$ pytest testing/review_test.py
$ pytest testing/association_proxy_test.py
$ pytest testing/serialization_test.py
$ pytest
```

![All tests passing](assets/pytest-results.png)

## Project layout

```
server/
  app.py        Flask app, database config, migrations
  models.py     Customer, Item, Review, and the Marshmallow schemas
  seed.py       Sample customers, items, and reviews
  testing/      Tests for the models, association proxy, and serialization
```

## Resources

- [Marshmallow documentation](https://marshmallow.readthedocs.io/en/stable/)
- [Flask-SQLAlchemy documentation](https://flask-sqlalchemy.palletsprojects.com/)
