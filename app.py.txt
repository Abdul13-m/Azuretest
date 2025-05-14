# app.py

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
import time

# Configuration
village_id = 'PK20301001001'
login_url = 'https://kf.kobo.iom.int/accounts/login/?next=%2F%23%2Flogin'
form_url = 'https://kf.kobo.iom.int/#/forms/awSuNUXMitvmg5cNoCcLDj/data/table'
username = 'pak_dtm_admin'
password = 'Ros9Pencil99'
column_number = 10  # Change this as needed

# Set up headless Chrome options
options = Options()
options.add_argument("--headless")
options.add_argument("--disable-gpu")
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")
options.add_argument("--window-size=1920x1080")

# Start WebDriver with WebDriver Manager
driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=options)

try:
    # Step 1: Open login page
    driver.get(login_url)

    WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.ID, 'id_login'))
    ).send_keys(username)

    driver.find_element(By.ID, 'id_password').send_keys(password)
    driver.find_element(By.XPATH, '//button[@type="submit" and @name="Login"]').click()

    WebDriverWait(driver, 10).until(
        EC.url_contains('/#/')
    )

    # Step 2: Open the form
    driver.get(form_url)

    # Step 3: Apply filter
    filter_inputs = WebDriverWait(driver, 15).until(
        EC.presence_of_all_elements_located((By.CLASS_NAME, 'table-filter-input'))
    )

    if column_number < len(filter_inputs):
        search_input = filter_inputs[column_number]
        search_input.clear()
        search_input.send_keys(village_id)
        search_input.send_keys(u'\ue007')  # Press Enter key

        WebDriverWait(driver, 15).until(
            EC.presence_of_element_located((By.XPATH, f"//*[contains(text(), '{village_id}')]"))
        )
        print(f"Village ID '{village_id}' found and filtered successfully.")
    else:
        print(f"Invalid column number: {column_number}")

except Exception as e:
    print(f"An error occurred: {e}")

finally:
    driver.quit()
