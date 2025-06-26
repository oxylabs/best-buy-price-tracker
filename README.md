# Best Buy Price Tracker

[![Oxylabs promo code](https://raw.githubusercontent.com/oxylabs/product-integrations/refs/heads/master/Affiliate-Universal-1090x275.png)](https://oxylabs.io/pages/gitoxy?utm_source=877&utm_medium=affiliate&groupid=877&utm_content=best-buy-price-tracker-github&transaction_id=102f49063ab94276ae8f116d224b67)

[![](https://dcbadge.vercel.app/api/server/eWsVUJrnG5)](https://discord.gg/Pds3gBmKMH)

Here, you'll find the process of building a scalable price tracker for Best Buy, one of the largest e-commerce websites for electronics.

The tutorial uses Python and Oxylabs’ [Best Buy API](https://oxylabs.io/products/scraper-api/ecommerce/bestbuy) (a part of Web Scraper API). You can get a **1-week free trial** by registering on the [dashboard](https://dashboard.oxylabs.io/).

For visualizations and in-depth explanations, see our [blog post](https://oxylabs.io/blog/best-buy-price-tracker). 

## 1. Installing prerequisite libraries

```python
pip install pandas
pip install matplotlib
```

## 2. Making the initial request

```python
import requests

USERNAME = "username"
PASSWORD = "password"

# Structure payload.
payload = {
   'source': 'universal',
   'url': "https://www.bestbuy.com/site/samsung-galaxy-z-flip4-128gb-unlocked-graphite/6512618.p?skuId=6512618&intl=nosplash",
   'geo_location': 'United States',
   'parse': True,
}

# Get response.
    response = requests.request(
        'POST',
        'https://realtime.oxylabs.io/v1/queries',
        auth=(USERNAME, PASSWORD),
        json=payload,
    )

print(response.json())
```

## 3. Creating the core of the tracker

Create a function that would read the historical price tracker data.

```python
def read_past_data(filepath):
    results = {}

    if not os.path.isfile(filepath):
        open(filepath, 'a').close()

    if not os.stat(filepath).st_size == 0:
        results_df = pd.read_json(filepath, convert_axes=False)
        results = results_df.to_dict()
        return results
    
    return results
```

As the historical price data is now loaded, think of a function that would take the past price tracker data and add the present price to it.

```python
def add_todays_prices(results, tracked_product_links):
    today = date.today()

    for link in tracked_product_links:
        product = get_product(link)

        if product["title"] not in results:
            results[product["title"]] = {}
        
        results[product["title"]][today.strftime("%d %B, %Y")] = {
            "price": product["price"],
            "currency": product["currency"],
        }
    
    return results
```

Having the prices updated for the present, move on to saving the results back to the file you started from, thus finishing the process loop.

```python
def save_results(results, filepath):
    df = pd.DataFrame.from_dict(results)

    df.to_json(filepath)

    return
```

Finally, move the connection to the Scraper API to a separate function and combine all you have so far.

```python
import os
import requests
import os.path
from datetime import date
import pandas as pd

def get_product(link):
    USERNAME = "username"
    PASSWORD = "password"

    # Structure payload.
    payload = {
        'source': 'universal',
        'url': link,
        'geo_location': 'United States',
        'parse': True,
    }

    # Get response.
    response = requests.request(
        'POST',
        'https://realtime.oxylabs.io/v1/queries',
        auth=(USERNAME, PASSWORD),
        json=payload,
    )
    response_json = response.json()

    content = response_json["results"][0]["content"]

    product = {
        "title": content["title"],
        "price": content["price"]["price"],
        "currency": content["price"]["currency"]
    }
    return product

def read_past_data(filepath):
    results = {}

    if not os.path.isfile(filepath):
        open(filepath, 'a').close()

    if not os.stat(filepath).st_size == 0:
        results_df = pd.read_json(filepath, convert_axes=False)
        results = results_df.to_dict()
        return results
    
    return results

def save_results(results, filepath):
    df = pd.DataFrame.from_dict(results)

    df.to_json(filepath)

    return

def add_todays_prices(results, tracked_product_links):
    today = date.today()

    for link in tracked_product_links:
        product = get_product(link)

        if product["title"] not in results:
            results[product["title"]] = {}
      
        results[product["title"]][today.strftime("%d %B, %Y")] = {
            "price": product["price"],
            "currency": product["currency"],
        }
  
    return results

def main():
    results_file = "data.json"

    tracked_product_links = [
        "https://www.bestbuy.com/site/samsung-galaxy-z-flip4-128gb-unlocked-graphite/6512618.p?skuId=6512618&intl=nosplash",
        "https://www.bestbuy.com/site/samsung-galaxy-z-flip5-256gb-unlocked-graphite/6548838.p?skuId=6548838"
    ]

    past_results = read_past_data(results_file)

    updated_results = add_todays_prices(past_results, tracked_product_links)

    save_results(updated_results, results_file)
  
if __name__ == "__main__":
    main()
```

## 4. Plotting price history

```python
def plot_history_chart(results):
    for product in results:
        dates = []
        prices = []
        
        for entry_date in results[product]:
            dates.append(entry_date)
            prices.append(results[product][entry_date]["price"])

        plt.plot(dates,prices, label=product)
        
        plt.xlabel("Date")
        plt.ylabel("Price")

    plt.title("Product prices over time")
    plt.legend()
    plt.show()
```

## 5. Creating price drop alerts

```python
def check_for_pricedrop(results):
    for product in results:
        today = date.today()
        yesterday = today - timedelta(days = 1)

        change = results[product][today.strftime("%d %B, %Y")]["price"] - results[product][yesterday.strftime("%d %B, %Y")]["price"]

        if change < 0:
            print(f'Price for {product} has dropped by {change}!')
```

## 6. The final code

```python
import os
import requests
import os.path
from datetime import date
from datetime import timedelta
import pandas as pd
import matplotlib.pyplot as plt


def get_product(link):
    USERNAME = "username"
    PASSWORD = "password"

    # Structure payload.
    payload = {
        'source': 'universal',
        'url': link,
        'geo_location': 'United States',
        'parse': True,
    }

    # Get response.
    response = requests.request(
        'POST',
        'https://realtime.oxylabs.io/v1/queries',
        auth=(USERNAME, PASSWORD),
        json=payload,
    )
    response_json = response.json()

    content = response_json["results"][0]["content"]

    product = {
        "title": content["title"],
        "price": content["price"]["price"],
        "currency": content["price"]["currency"]
    }
    return product

def read_past_data(filepath):
    results = {}

    if not os.path.isfile(filepath):
        open(filepath, 'a').close()

    if not os.stat(filepath).st_size == 0:
        results_df = pd.read_json(filepath, convert_axes=False)
        results = results_df.to_dict()
        return results
    
    return results

def save_results(results, filepath):
    df = pd.DataFrame.from_dict(results)

    df.to_json(filepath)

    return

def add_todays_prices(results, tracked_product_links):
    today = date.today()

    for link in tracked_product_links:
        product = get_product(link)

        if product["title"] not in results:
            results[product["title"]] = {}
        
        results[product["title"]][today.strftime("%d %B, %Y")] = {
            "price": product["price"],
            "currency": product["currency"],
        }
    
    return results

def plot_history_chart(results):
    for product in results:
        dates = []
        prices = []
        
        for entry_date in results[product]:
            dates.append(entry_date)
            prices.append(results[product][entry_date]["price"])

        plt.plot(dates,prices, label=product)
        
        plt.xlabel("Date")
        plt.ylabel("Price")

    plt.title("Product prices over time")
    plt.legend()
    plt.show()

def check_for_pricedrop(results):
    for product in results:
        today = date.today()
        yesterday = today - timedelta(days = 1)

        change = results[product][today.strftime("%d %B, %Y")]["price"] - results[product][yesterday.strftime("%d %B, %Y")]["price"]

        if change < 0:
            print(f'Price for {product} has dropped by {change}!')


def main():
    results_file = "data.json"

    tracked_product_links = [
        "https://www.bestbuy.com/site/samsung-galaxy-z-flip4-128gb-unlocked-graphite/6512618.p?skuId=6512618&intl=nosplash",
        "https://www.bestbuy.com/site/samsung-galaxy-z-flip5-256gb-unlocked-graphite/6548838.p?skuId=6548838"
    ]

    past_results = read_past_data(results_file)

    updated_results = add_todays_prices(past_results, tracked_product_links)

    plot_history_chart(updated_results)

    check_for_pricedrop(updated_results)

    save_results(updated_results, results_file)
    
if __name__ == "__main__":
    main()
```

## Wrapping up

For all of the API parameters, see our [documentation](https://developers.oxylabs.io/scraper-apis/web-scraper-api/best-buy).

If you need assistance, don't hesitate to contact us at support@oxylabs.io.
