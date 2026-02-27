# 📚 학습 기록 저장소 (dobong_study)

### 1. 학습 요약
웹 페이지의 성격(정적 vs 동적)에 따라 최적의 도구(**BeautifulSoup** 또는 **Playwright**)를 선택하여 데이터를 수집하는 전체적인 웹 스크래핑 체계를 학습함.

### 2. 배운 개념 정리
* **정적 스크래핑 (BeautifulSoup):** * 서버가 준 HTML 문서를 그대로 읽는 방식.
    * 속도가 빠르고 가볍지만, 자바스크립트로 나중에 생성되는 데이터는 가져올 수 없음.
* **동적 스크래핑 (Playwright):** * 실제 브라우저를 띄워 자바스크립트까지 실행시킨 뒤 데이터를 가져오는 방식.
    * 스크롤, 버튼 클릭 등이 필요한 사이트에 필수적이지만 리소스를 많이 소모함.



### 3. 코드리뷰
```python
import requests
from bs4 import BeautifulSoup
from playwright.sync_api import sync_playwright

# [방법 1] 정적 스크래핑: 가볍고 빠른 수집
def scrape_static(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.text, "html.parser")
    # 설계 의도: 데이터가 HTML에 고스란히 담겨 있을 때 효율적으로 추출
    return [a.get_text() for a in soup.select("a.gPFEn")]

# [방법 2] 동적 스크래핑: 브라우저 제어를 통한 수집
def scrape_dynamic(url):
    with sync_playwright() as p:
        # 설계 의도: 자바스크립트 로딩이나 사용자 동작이 필요한 경우 브라우저 직접 제어
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()
        page.goto(url)
        page.wait_for_selector("a.gPFEn") # 데이터가 나타날 때까지 대기
        
        titles = [a.inner_text() for a in page.query_selector_all("a.gPFEn")]
        browser.close()
        return titles

url = "[https://news.google.com/](https://news.google.com/)..."
# 상황에 맞춰 도구를 선택해서 사용함
```
**[설계 의도 및 리뷰]**
* **상황별 도구 선택:** 무조건 최신 기술(Playwright)만 쓰는 것이 아니라, 사이트 구조에 따라 `requests`를 써서 서버 부하를 줄이는 '실무적 판단'을 코드로 구현함.
* **Wait 로직:** 동적 스크래핑 시 `wait_for_selector`를 통해 데이터 누락을 방지하는 안정성을 확보함.

### 4. 헷갈렸던 점
* **Q: 두 가지 방식 중 무엇을 먼저 시도해야 하나요?**
    * **A:** 무조건 **정적 방식(BeautifulSoup)**을 먼저 시도해 보세요! 만약 데이터가 안 불려온다면 그때 **동적 방식(Playwright)**으로 전환하는 게 실무 효율 면에서 훨씬 좋습니다.

### 5. 실무 관점
* **자원 및 비용:** 실제 서비스 운영 시 Playwright는 CPU와 메모리를 많이 잡아먹습니다. 수천 개의 페이지를 긁어야 한다면 인프라 비용 문제를 고려해 정적 스크래핑이나 API를 우선순위에 둡니다.
* **차단 방지:** 두 방식 모두 너무 빠른 요청은 차단(IP Ban) 사유가 되므로 `time.sleep`이나 Playwright의 대기 기능을 적절히 섞어 사람처럼 행동하는 것이 중요합니다.