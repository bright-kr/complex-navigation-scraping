# 복잡한 내비게이션이 있는 웹사이트 스크레이핑

[![Promo](https://github.com/bright-kr/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/) 

이 가이드는 동적 페이지네이션, 무한 스크롤, ‘Load More’ 버튼과 같은 복잡한 내비게이션 패턴을 가진 웹사이트를 Selenium 및 브라우저 자동화를 사용하여 스크레이핑하는 방법을 설명합니다.

- [복잡한 내비게이션으로 간주되는 것은 무엇입니까?](#what-is-considered-complex-navigation)
- [복잡한 내비게이션 웹사이트를 처리하기 위한 도구](#tools-to-handle-complex-navigation-websites)
- [일반적인 복잡한 내비게이션 패턴 스크레이핑](#scraping-common-complex-navigation-patterns)
  - [동적 페이지네이션](#dynamic-pagination)
  - [‘Load More’ 버튼](#load-more-button)
  - [무한 스크롤](#infinite-scrolling)
- [결론](#conclusion)

## What Is Considered Complex Navigation?

Web스크레이핑에서 복잡한 내비게이션이란 콘텐츠나 페이지에 쉽게 접근할 수 없는 웹사이트 구조를 의미합니다. 복잡한 내비게이션 시나리오는 종종 동적 요소, 비동기 데이터 로딩 또는 사용자 주도 상호작용을 포함합니다. 이러한 측면은 사용자 경험을 향상시킬 수 있지만, 데이터 추출을 크게 복잡하게 만들기도 합니다. 다음은 매우 일반적인 예시입니다:

- **JavaScript-rendered navigation**: JavaScript 프레임워크에 의존하여 브라우저에서 직접 콘텐츠를 생성하는 웹사이트입니다.
- **Paginated content**: AJAX를 통해 페이지네이션이 동적으로 로드되며 데이터가 여러 페이지에 걸쳐 분산된 사이트입니다.
- **Infinite scrolling**: 사용자가 스크롤할 때 추가 콘텐츠를 동적으로 로드하는 페이지로, 소셜 미디어 피드, Discourse 기반 포럼, 뉴스 웹사이트에서 흔합니다.
- **Multi-level menus**: 더 깊은 내비게이션 계층을 표시하기 위해 여러 번 클릭하거나 hover 동작이 필요한 중첩 메뉴를 가진 사이트로, 마켓플레이스의 제품 카테고리 트리에서 흔합니다.
- **Interactive map interfaces**: 지도나 그래프에 데이터를 표시하는 웹사이트로, 사용자가 이동(pan)하거나 확대/축소(zoom)할 때 정보가 동적으로 로드됩니다.
- **Tabs or accordions**: 서버가 반환한 HTML에 직접 포함되어 있지 않고, 동적으로 렌더링된 탭 또는 접을 수 있는 아코디언 아래에 콘텐츠가 숨겨진 페이지입니다.
- **Dynamic filters and sorting options**: 여러 필터를 적용하면 URL 구조를 변경하지 않고도 항목 목록이 동적으로 다시 로드되는 복잡한 필터링 시스템을 가진 사이트입니다.

## Tools to Handle Complex Navigation Websites

위에 나열된 복잡한 상호작용의 대부분은 JavaScript 실행이 필요하며, 이는 브라우저만 수행할 수 있습니다. 즉, 이러한 페이지에서는 단순한 [HTML parsers](https://brightdata.co.kr/blog/web-data/best-html-parsers)에 의존할 수 없습니다. 대신 Selenium, Playwright 또는 Puppeteer와 같은 브라우저 자동화 도구를 사용해야 합니다. 이러한 솔루션을 사용하면 웹페이지에서 특정 작업을 수행하도록 브라우저에 프로그래밍 방식으로 지시하여 사용자 행동을 모방할 수 있습니다.

## Scraping Common Complex Navigation Patterns

이 가이드는 다음 세 가지 특정 유형의 복잡한 내비게이션 패턴을 다룹니다:

- **Dynamic pagination**: AJAX를 통해 페이지네이션 데이터가 동적으로 로드되는 사이트입니다.
- **‘Load More’ button**: 일반적인 JavaScript 기반 내비게이션 예시입니다.
- **Infinite scrolling**: 사용자가 아래로 스크롤함에 따라 지속적으로 데이터를 로드하는 페이지입니다.

Python에서 Selenium을 사용하지만, 로직은 Playwright, Puppeteer 또는 기타 브라우저 자동화 도구에도 적용할 수 있습니다. 또한 이 가이드는 사용자가 이미 [web scraping using Selenium](https://brightdata.co.kr/blog/how-tos/using-selenium-for-web-scraping)의 기본에 익숙하다고 가정합니다.

### Dynamic Pagination

“[Oscar Winning Films: AJAX and Javascript](https://www.scrapethissite.com/pages/ajax-javascript/#2014)” 스크레이핑 샌드박스를 사용하겠습니다:

![The target page. Note how pagination data is loaded dynamically](https://github.com/bright-kr/complex-navigation-scraping/blob/main/Images/Dynamic-pagniation-example-1536x752.gif)

이 사이트는 오스카 수상 영화 데이터를 연도별로 페이지네이션하여 동적으로 로드합니다.

이와 같은 페이지를 효과적으로 탐색하고 스크레이핑하려면 다음 단계를 따라야 합니다:

1. 새 연도를 클릭하여 데이터 로딩을 트리거합니다(로더 요소가 표시됩니다).
2. 로더 요소가 사라질 때까지 기다립니다(데이터가 완전히 로드되었음을 의미합니다).
3. 데이터가 포함된 테이블이 페이지에 제대로 렌더링되었는지 확인합니다.
4. 데이터가 사용 가능해지면 스크레이핑합니다.

아래는 Python에서 Selenium으로 이 로직을 구현하는 예시입니다:

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options

# Set up Chrome options for headless mode
options = Options()
options.add_argument("--headless")

# Create a Chrome web driver instance
driver = webdriver.Chrome(service=Service(), options=options)

# Connect to the target page
driver.get("https://www.scrapethissite.com/pages/ajax-javascript/")

# Click the "2012" pagination button
element = driver.find_element(By.ID, "2012")
element.click()

# Wait until the loader is no longer visible
WebDriverWait(driver, 10).until(
    lambda d: d.find_element(By.CSS_SELECTOR, "#loading").get_attribute("style") == "display: none;"
)

# Data should now be loaded...

# Wait for the table to be present on the page
WebDriverWait(driver, 10).until(
    EC.presence_of_element_located((By.CSS_SELECTOR, ".table"))
)

# Where to store the scraped data
films = []

# Scrape data from the table
table_body = driver.find_element(By.CSS_SELECTOR, "#table-body")
rows = table_body.find_elements(By.CSS_SELECTOR, ".film")
for row in rows:
    title = row.find_element(By.CSS_SELECTOR, ".film-title").text
    nominations = row.find_element(By.CSS_SELECTOR, ".film-nominations").text
    awards = row.find_element(By.CSS_SELECTOR, ".film-awards").text
    best_picture_icon = row.find_element(By.CSS_SELECTOR, ".film-best-picture").find_elements(By.TAG_NAME, "i")
    best_picture = True if best_picture_icon else False

    # Store the scraped data
    films.append({
      "title": title,
      "nominations": nominations,
      "awards": awards,
      "best_picture": best_picture
    })

# Data export logic...

# Close the browser driver
driver.quit()
```

해당 코드 스니펫의 구성은 다음과 같습니다:

1.  코드는 headless Chrome 인스턴스를 설정합니다.
2.  스크립트는 대상 페이지를 열고 “2012” 페이지네이션 버튼을 클릭하여 데이터 로딩을 트리거합니다.
3.  Selenium은 [`WebDriverWait()`](https://selenium-python.readthedocs.io/waits.html)을 사용하여 로더가 사라질 때까지 대기합니다.
4.  로더가 사라진 후, 스크립트는 테이블이 나타날 때까지 대기합니다.
5.  데이터가 완전히 로드된 후, 스크립트는 영화 제목, 후보 수, 수상 수, 작품상 수상 여부와 같은 세부 정보를 추출합니다. 추출된 정보는 딕셔너리 목록에 저장됩니다.

결과는 다음과 같습니다:

```json
[
  {
    "title": "Argo",
    "nominations": "7",
    "awards": "3",
    "best_picture": true
  },
  // ...
  {
    "title": "Curfew",
    "nominations": "1",
    "awards": "1",
    "best_picture": false
  }
]
```

이 내비게이션 패턴을 처리하는 단 하나의 최선의 접근법이 항상 존재하는 것은 아니라는 점을 유념하시기 바랍니다. 페이지의 동작에 따라서는 대체 방법이 필요할 수 있습니다. 다음은 몇 가지 예시입니다:

*   `WebDriverWait()`을 예상 조건과 함께 사용하여 특정 HTML 요소가 나타나거나 사라질 때까지 대기합니다.
*   AJAX 요청 트래픽을 모니터링하여 새 콘텐츠가 언제 가져와지는지 감지합니다. 이는 브라우저 로깅을 사용하는 방식일 수 있습니다.
*   페이지네이션에 의해 트리거되는 API 요청를 식별하고, 직접 요청를 보내 프로그래밍 방식으로 데이터를 가져옵니다(예: [`requests` library](https://brightdata.co.kr/blog/web-data/python-requests-guide) 사용).

### ‘Load More’ Button

사용자 상호작용이 포함된 JavaScript 기반 복잡한 내비게이션 시나리오를 설명하기 위해 'Load More' 버튼 예시를 사용하겠습니다. 개념은 단순합니다. 항목 목록이 표시되고, 버튼을 클릭하면 추가 항목이 로드됩니다.

이번에는 Scraping Course의 [‘Load More’ example](https://www.scrapingcourse.com/button-click) 페이지를 대상 사이트로 사용하겠습니다:

![The ‘Load More’ target page in action](https://github.com/bright-kr/complex-navigation-scraping/blob/main/Images/Clicking-on-the-load-more-button-1536x752.gif)

이 복잡한 내비게이션 스크레이핑 패턴을 처리하려면 다음 단계를 따르십시오:

1.  ‘Load More’ 버튼을 찾아 클릭합니다.
2.  새 요소가 페이지에 로드될 때까지 기다립니다.

다음은 Selenium에서 사용할 코드입니다:

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait

# Set up Chrome options for headless mode
options = Options()
options.add_argument("--headless")

# Create a Chrome web driver instance
driver = webdriver.Chrome(options=options)

# Connect to the target page
driver.get("https://www.scrapingcourse.com/button-click")

# Collect the initial number of products
initial_product_count = len(driver.find_elements(By.CSS_SELECTOR, ".product-item"))

# Locate the "Load More" button and click it
load_more_button = driver.find_element(By.CSS_SELECTOR, "#load-more-btn")
load_more_button.click()

# Wait until the number of product items on the page has increased
WebDriverWait(driver, 10).until(lambda driver: len(driver.find_elements(By.CSS_SELECTOR, ".product-item")) > initial_product_count)

# Where to store the scraped data
products = []

# Scrape product details
product_elements = driver.find_elements(By.CSS_SELECTOR, ".product-item")
for product_element in product_elements:
    # Extract product details
    name = product_element.find_element(By.CSS_SELECTOR, ".product-name").text
    image = product_element.find_element(By.CSS_SELECTOR, ".product-image").get_attribute("src")
    price = product_element.find_element(By.CSS_SELECTOR, ".product-price").text
    url = product_element.find_element(By.CSS_SELECTOR, "a").get_attribute("href")

    # Store the scraped data
    products.append({
        "name": name,
        "image": image,
        "price": price,
        "url": url
    })

# Data export logic...

# Close the browser driver
driver.quit()
```

'Load More' 버튼 내비게이션 패턴을 처리하기 위해 스크립트는 다음을 수행합니다:

1.  페이지의 초기 제품 수를 기록합니다
2.  “Load More” 버튼을 클릭합니다
3.  제품 수가 증가할 때까지 대기하여 새 항목이 추가되었음을 확인합니다

이 접근법은 로드될 요소의 정확한 개수를 알 필요가 없으므로 효율적이면서도 범용적입니다. 다만, 대체 방법으로도 유사한 결과를 얻을 수 있습니다.

### Infinite Scrolling

무한 스크롤은 사용자 참여를 높이기 위해 소셜 미디어 및 이커머스 플랫폼에서 널리 사용되는 인기 있는 상호작용입니다. 이 경우 대상은 위와 동일한 페이지이되, [‘Load More’ 버튼 대신 무한 스크롤](https://www.scrapingcourse.com/infinite-scrolling)을 사용하는 버전입니다:

![infinite scrolling instead of a 'Load More' button](https://github.com/bright-kr/complex-navigation-scraping/blob/main/Images/Infinite-scrolling-example-1024x501.gif)

대부분의 브라우저 자동화 도구는 페이지를 위/아래로 스크롤하는 직접적인 메서드를 제공하지 않으며, Selenium도 예외가 아닙니다. 대신 스크롤 동작을 수행하려면 페이지에서 JavaScript 스크립트를 실행해야 합니다.

해결책은 아래로 스크롤하는 사용자 정의 JavaScript 스크립트를 작성하는 것입니다:

1.  지정된 횟수만큼, 또는
2.  더 이상 로드할 데이터가 없을 때까지.

> **Note**:\
> 각 스크롤은 새 데이터를 로드하며 페이지의 요소 개수를 증가시킵니다.

그 다음 새로 로드된 콘텐츠를 스크레이핑할 수 있습니다.

다음은 Selenium에서 무한 스크롤을 사용하는 코드입니다:

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait

# Set up Chrome options for headless mode
options = Options()
# options.add_argument("--headless")

# Create a Chrome web driver instance
driver = webdriver.Chrome(options=options)

# Connect to the target page with infinite scrolling
driver.get("https://www.scrapingcourse.com/infinite-scrolling")

# Current page height
scroll_height = driver.execute_script("return document.body.scrollHeight")
# Number of products on the page
product_count = len(driver.find_elements(By.CSS_SELECTOR, ".product-item"))

# Max number of scrolls
max_scrolls = 10
scroll_count = 1

# Limit the number of scrolls to 10
while scroll_count < max_scrolls:
    # Scroll down
    driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")

    # Wait until the number of product items on the page has increased
    WebDriverWait(driver, 10).until(lambda driver: len(driver.find_elements(By.CSS_SELECTOR, ".product-item")) > product_count)

    # Update the product count
    product_count = len(driver.find_elements(By.CSS_SELECTOR, ".product-item"))

    # Get the new page height
    new_scroll_height = driver.execute_script("return document.body.scrollHeight")

    # If no new content has been loaded
    if new_scroll_height == scroll_height:
        break

    # Update scroll height and increment scroll count
    scroll_height = new_scroll_height
    scroll_count += 1

# Scrape product details after infinite scrolling
products = []
product_elements = driver.find_elements(By.CSS_SELECTOR, ".product-item")
for product_element in product_elements:
    # Extract product details
    name = product_element.find_element(By.CSS_SELECTOR, ".product-name").text
    image = product_element.find_element(By.CSS_SELECTOR, ".product-image").get_attribute("src")
    price = product_element.find_element(By.CSS_SELECTOR, ".product-price").text
    url = product_element.find_element(By.CSS_SELECTOR, "a").get_attribute("href")

    # Store the scraped data
    products.append({
        "name": name,
        "image": image,
        "price": price,
        "url": url
    })

# Export to CSV/JSON...

# Close the browser driver
driver.quit() 
```

이 스크립트는 먼저 현재 페이지 높이와 제품 수를 식별하여 무한 스크롤을 처리합니다. 스크롤 프로세스를 최대 10회 반복으로 제한합니다. 각 반복에서 다음을 수행합니다:

1.  하단까지 스크롤합니다
2.  제품 수가 증가할 때까지 대기합니다(새 콘텐츠가 로드되었음을 의미합니다)
3.  추가 콘텐츠의 존재 여부를 감지하기 위해 페이지 높이를 비교합니다

스크롤 이후 페이지 높이가 변하지 않으면, 더 이상 로드할 데이터가 없다는 신호로 루프가 종료됩니다.

## Conclusion

복잡한 내비게이션 패턴이 포함되면 Web스크레이핑은 어려울 수 있으며, 기업은 자동화 스크립트를 차단하기 위해 anti-scraping 조치를 적용하여 이를 더욱 어렵게 만들 수 있습니다. Selenium과 같은 브라우저 자동화 도구는 이러한 제한을 우회할 수 없습니다.

해결책은 Playwright, Puppeteer, Selenium 및 기타 도구와 통합되며 각 요청마다 IP를 자동으로 로ーテーティング프록시하는 [Scraping Browser](https://brightdata.co.kr/products/scraping-browser)와 같은 클라우드 기반 브라우저를 사용하는 것입니다. 이는 브라우ザフィンガープリント, 재시도, CAPTCHA 해결 등을 관리할 수 있습니다. 복잡한 사이트를 탐색할 때 차단되는 문제와 작별하십시오!