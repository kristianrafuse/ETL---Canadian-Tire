# ETL---Canadian-Tire
This ETL process scrapes the CanadianTire website for clearance and sale products, cleans and organizes the data, and then loads them to a SQL database. I subsequently access this database with a web app in another repo.

**ETL**
-------
* re
* os
* time
* math
* pandas
* urlparse
* sqlalchemy
* BeautifulSoup
* splinter/Browser
* selenium webdriver

**ETL Highlights**
-------

```
    chrome_options = Options()
    chrome_options.add_argument("--incognito")
    chrome_options.add_experimental_option("prefs", {
        "profile.default_content_setting_values.geolocation": 1
})
```

I wanted to dynamically determine the number of loops to use. I judged that total results, or total number of products on sale, divided by the default number of products per page worked well. Using math.ceil to always round up. 
```
    total_results_element = html_soup.find('span', class_='nl-filters__results')
    total_results_text = total_results_element.text if total_results_element else '0'

Use re to extract the first sequence of digits found in the total_results_text and store it in the variable total_results as an integer.

    total_results = int(re.search(r'\d+', total_results_text).group())

Default number of products per page

    items_per_page = 24
    total_pages = math.ceil(total_results / items_per_page)
```

Basic structure of locating the object then scraping out the details 
```  
    original_price_element = product.find('s', attrs={'aria-hidden': 'true'})
    original_price = original_price_element.text.strip() if original_price_element else 'N/A'

There were some products with prices in different places, so this fills in those spaces. 

    if original_price == 'N/A':
    alt_original_price_element = product.find('span', class_='nl-price--total')
    original_price = alt_original_price_element.text.strip() if alt_original_price_element else 'N/A'

I notice that the urls of the images links contained product category information.
    img_tag = product.find('img', attrs={'data-src': True})
    product_image_link = img_tag['data-src'] if img_tag and 'data-src' in img_tag.attrs else 'N/A'

```

I wasn't actually interested in the image links as image links, but rather to parse out product category information, which didn't seem to be present anywhere else on the page. I created a function that uses urllib.parse from urlparse to grab the information that I wanted. I was having trouble getting re to work here, and this was simpler.
```
    def extract_category(link_url):
        parsed_url = urlparse(link_url)
        path_components = parsed_url.path.split('/')
        category = '/'.join(path_components[3:4])
        return category
```

handle survey pop-up
```
    try:
        not_right_now_button = WebDriverWait(driver3, 10).until(
            EC.element_to_be_clickable((By.ID, "kplDeclineButton"))
        )
        not_right_now_button.click()
    except:
        pass
```

Error handling example that should have been taken care of earlier, I think.
```
    sales_data_to_insert = []
    for row in csv_reader:
        original_price = float(row[1]) if row[1] != '' else None
        clearance_price = float(row[2]) if row[2] != '' else None
        percentage_off = float(row[3]) if row[3] != '' else None
        rating = float(row[6]) if row[6] != '' else None

        sales_data_to_insert.append({
            "product_name": row[0],
            "original_price": original_price,
            "clearance_price": clearance_price,
            "percentage_off": percentage_off,
            "product_category": row[4],
            "product_code": row[5],
            "rating": rating,
            "link": row[7]
        })

Insert the data into the "sales" table
    sales_insert_query = sales_table.insert()
    engine.execute(sales_insert_query, sales_data_to_insert)
    engine.dispose()
