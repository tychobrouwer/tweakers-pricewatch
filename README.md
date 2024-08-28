# Tweakers Pricewatch

The goal is to create a web application that can keep track of the prices of products on Tweakers and send email notifications when the price of a product drops below a certain threshold.

Products can be added to the watchlist by entering the URL or product id of the product on Tweakers. The application will then scrape the product page for the price and name of the product.

## Tweakers API

### Price History

Product price history can be retrieved from the Tweakers API. The API is not public, but can be accessed by sending a GET request to the following URL:

```https://tweakers.net/ajax/price_chart/{product_id}/nl/```

The response will be a JSON object containing the price history of the product.

```jsonc
{
    "dataset": {
        "source": [
            [
                "2023-09-16",       // Timestamp
                289,                // Minimum price
                289                 // Average price
            ],
            // ...
        ],
        "dimensions": [             // Dimensions of the dataset within the source array
            "timestamp",
            "minimum",
            "average"
        ]
    },
    "markers": [                    // Markers on the chart
        [
            {
                "yAxis": 219,
                "name": " Laagste prijs sinds 6 maanden"
            }
        ]
    ],
    "success": true                 // Indicates if the request was successful
}
```

### Store Prices

Web scraping can be used to retrieve the current price of a product from the Tweakers website. The price can be found in the HTML of the product page.

```html
<table class="shop-listing"> <!-- Table with class "shop-listing" without class containing "refurbished" -->
    <tbody>
        <tr data-shop-id="7638"> <!-- Row for shop -> tr tags in table with "shoplisting" class -->
            <td class="shop-name">
                <p class="ellipsis">
                    <a>Belsimpel</a> <!-- Shop name -> select a tag -->
                    <span class="tagline"> Voor 23:30u besteld = morgen gratis in huis!</span>
                </p>
            </td>
            <td class="shop-score withReviewCount">
                <!-- Shop rating -->
            </td>
            <td class="shop-delivery">
                <!-- Delivery times of shop -->
            </td>
            <td class="shop-bare-price">
                <p>
                    <a>€&nbsp;233,95</a> <!-- Price of product -> select a tag -->
                </p>
            </td>
            <td class="shop-price">
                <p>
                    <a>€&nbsp;233,95</a> <!-- Price of product with shipping -> select a tag -->
                </p>
            </td>
            <td class="shop-clickout">
                <p>
                    <a href="https://tweakers.net/pricewatch/clickout?shop-id=3695&pe-id=
                    1978222&link=5960304f93e94fa794a5fadcae7fdddc512887793fff218b206c5ebf
                    203085a47e252cfe7bf3291c8675efd6314a924c3fba6a474148634d6548062941130
                    5554c4ce8582289d1e2361a68bba41c260e7da92854782f9e2f6d25d8fd566bee3884
                    1cf7b30ee77f11771c8fe81086026d5bc58916519e19cd6aad2fafe73717a5ef060b9
                    46f9b35f2d47f7761c06e99db4e36bbbfb9f5c60739eee94837c8ec2f543773c65702
                    a2386a9c1cfee842e6052922b4fba6b0b26eb15110d402f725e75d5e263a3ad72249d
                    7f1f1d0c9c068f8dfeddd755f36eb4c0fb0910488c4ee4609659281abc6142719ef0b
                    a9cdbd2374fb7a3097d75b9459db362edb&nonce=9f420f306cb8c195a80640c6798a
                    2dff407d99d7a09ff4a0">
                        <!-- Clickout button to shop -->
                    </a>
                </p>
            </td>
        </tr>
        <!-- ... -->
    </tbody>
</table>
```

For the clickout link, the link is an affiliate link that redirects to the shop's website. To get the actual link of the product, the link in the href attribute of the a tag can be requested. The response is an 302 redirect where the location header contains the actual link in the parameters which can be regexed.

Redirect link example:

```https://www.awin1.com/cread.php?v=8373&r=329213&p=https%3A%2F%2Fwww.mobiel.nl%2Faccessoires%2Fapple%2Fairpods-pro-2%2Fusb-c%3Futm_source%3Ddynamic%26utm_medium%3Daffils&clickref=xid:fr1724877900222adj```

Final link (regex https and remove parameters):

```https://www.mobiel.nl/accessoires/apple/airpods-pro-2/usb-c```
