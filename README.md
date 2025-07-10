# A2-creations
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from webdriver_manager.chrome import ChromeDriverManager
import pandas as pd
import time


def scrape_alibaba_rfq(url):
    options = webdriver.ChromeOptions()
    options.add_argument("--headless")  # Run headless Chrome
    driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=options)

    driver.get(url)
    wait = WebDriverWait(driver, 15)

    all_data = []

    while True:
        try:
            # Wait for RFQ list container to appear
            wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, ".rfq-list-wrap")))

            # Locate all RFQ items, adjust selector after inspecting page
            rfq_items = driver.find_elements(By.CSS_SELECTOR, ".rfq-list-wrap .rfq-item")

            for item in rfq_items:
                try:
                    # Customize selectors by inspecting target webpage
                    title = item.find_element(By.CSS_SELECTOR, ".rfq-title").text.strip()
                    quantity = item.find_element(By.CSS_SELECTOR, ".rfq-quantity").text.strip()
                    country = item.find_element(By.CSS_SELECTOR, ".rfq-country").text.strip()
                    date_posted = item.find_element(By.CSS_SELECTOR, ".rfq-date").text.strip()

                    rfq_data = {
                        "Title": title,
                        "Quantity": quantity,
                        "Country": country,
                        "DatePosted": date_posted,
                    }
                    all_data.append(rfq_data)
                except Exception as e:
                    print(f"Error parsing item: {e}")

            # Pagination: try clicking "next" button, else break
            next_buttons = driver.find_elements(By.CSS_SELECTOR, ".pagination-next")
            if next_buttons and next_buttons[0].is_enabled():
                next_buttons[0].click()
                time.sleep(3)  # Wait for page change
            else:
                break  # No next page, exit loop

        except Exception as e:
            print(f"Error during scraping: {e}")
            break

    driver.quit()

    # Save to CSV
    df = pd.DataFrame(all_data)
    df.to_csv("alibaba_rfq_data.csv", index=False, encoding="utf-8-sig")
    print("Scraping complete, data saved to alibaba_rfq_data.csv")

if __name__ == "__main__":
    target_url = "https://sourcing.alibaba.com/rfq/rfq_search_list.htm?spm=a2700.8073608.1998677541.1.82be65aaoUUItC&country=AE&recently=Y&tracelog=newest"
    scrape_alibaba_rfq(target_url)
