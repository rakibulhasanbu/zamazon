# Recommendation Service

**Technology:** Django
**Port:** 8201
**Priority:** P2 — Enhanced Buyer Experience

---

## Why Django

Recommendation engines are fundamentally a data science problem — they require statistical modelling, matrix factorisation, similarity scoring, and trend analysis across large datasets of buyer behaviour. Python's ecosystem (scikit-learn, pandas, NumPy, and SciPy) is the gold standard for this type of work and integrates naturally with Django. No other framework in this stack can match the breadth of Python's machine learning libraries. Building collaborative filtering in Go or NestJS would mean writing the algorithms from scratch; in Python they are available, tested, and battle-hardened.

---

## What This Service Does

- Generates personalised product recommendations for each buyer based on their purchase history, browsing history, wishlist, and search behaviour
- Powers the "Customers also bought" section on product pages using collaborative filtering — buyers with similar purchase patterns tend to like the same products
- Powers the "Related products" section using content-based filtering — products in the same category with similar attributes and price range
- Shows each buyer their recently viewed products in a persistent, cross-device history panel
- Generates trending product lists — products gaining purchase velocity in a given time window, used on the homepage and category pages
- Personalises the homepage product feed for logged-in buyers based on their interests and past behaviour
- Provides cold-start recommendations for new buyers with no history — falls back to trending and top-rated products by category
- Continuously retrains its models as new purchase and behaviour data flows in, so recommendations stay relevant

---

## Who Uses It

- **Buyers** — see personalised recommendations transparently throughout their browsing experience
- **customer-web** — displays recommendation widgets on the homepage, product pages, and cart page
- **search-service** — uses similarity data to power related product lookups

---

## Key Integrations

- Reads purchase history from order-service to build buyer preference models
- Reads recently viewed data from user-service
- Reads product metadata (categories, attributes, price) from product-service to compute content similarity
- Reads review ratings from review-service as a quality signal in recommendations
