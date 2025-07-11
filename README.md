# Python-da-Resmi-Gazete-den-Veri-ekip-Excel-e-Aktarma
Resmi Gazete sitesindeki son 1 ay verilerini Selenium aracılığıyla çekip Excel dosyasına kaydeden program

'''
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait, Select
from selenium.webdriver.support import expected_conditions as EC
import pandas as pd
import time

chrome_driver_yolu = "C:\\Users\\mehme\\Desktop\\Akıllı Sistemler Ödev\\chromedriver.exe"
service = Service(chrome_driver_yolu)
driver = webdriver.Chrome(service=service)

try:
    driver.get("https://www.resmigazete.gov.tr/")
    bekle = WebDriverWait(driver, 30)

    # Tarih aralığını seç
    select_elem = bekle.until(EC.presence_of_element_located((By.ID, "selectTarihAraligi")))
    select = Select(select_elem)
    select.select_by_value("son1ay")
    time.sleep(2)

    veri = []
    sayfa_no = 1
    max_sayfa = 50
    sonraki_sayfa_var = True

    while sonraki_sayfa_var and sayfa_no <= max_sayfa:
        print(f"\n{'='*50}")
        print(f"İşlenen sayfa: {sayfa_no}")
        
        # Tablonun tam olarak yüklenmesini bekle
        bekle.until(EC.presence_of_element_located((By.CSS_SELECTOR, "table tbody tr")))
        time.sleep(1)
        
        # Tüm satırları yeniden al
        rows = driver.find_elements(By.CSS_SELECTOR, "table tbody tr")
        print(f"Sayfada {len(rows)} satır bulundu")


        for i, row in enumerate(rows):

            if not row.is_displayed():
                continue
                
            cols = row.find_elements(By.TAG_NAME, "td")
            
            # 5 sütundan az olan veya boş satırları atla
            if len(cols) < 5 or not any(col.text.strip() for col in cols):
                continue

            try:
                # Başlık ve Mevzuat Türü
                baslik = cols[0].text.strip()
                
                mevzuat_turu = ""
                try:
                    mevzuat_div = cols[0].find_element(By.CSS_SELECTOR, "div.mt-1.small.text-secondary")
                    metin = mevzuat_div.text.strip()
                    if "Mevzuat Türü:" in metin:
                        mevzuat_turu = metin.split("Mevzuat Türü:")[-1].strip()
                except:
                    pass

                # Diğer bilgiler
                karar_no = cols[1].text.strip() if cols[1].text.strip() else ""
                tarih = cols[2].text.strip() if cols[2].text.strip() else ""
                sayisi = cols[3].text.strip() if cols[3].text.strip() else ""
                mukerrer = cols[4].text.strip() if cols[4].text.strip() else ""

                # Link
                link = ""
                try:
                    link_elem = cols[0].find_element(By.CSS_SELECTOR, "a[href]")
                    link = link_elem.get_attribute("href")
                except:
                    pass

                # Boş kayıtları filtrele
                if baslik or karar_no or tarih or sayisi:
                    veri.append({
                        "Başlık": baslik,
                        "Karar No": karar_no,
                        "Tarih": tarih,
                        "Sayısı": sayisi,
                        "Mükerrer": mukerrer,
                        "Mevzuat Türü": mevzuat_turu,
                        "Link": link
                    })
                    print(f"  - Kayıt {i+1}: {baslik[:50]}...")
            except Exception as e:
                print(f"Satır {i+1} işleme hatası: {str(e)}")
                continue

        try:

            sayfalandirma = driver.find_element(By.CSS_SELECTOR, "ul.sayfalandirma")
            next_buttons = sayfalandirma.find_elements(By.XPATH, ".//li[contains(@class, 'page-item')]")
            
            # Sonraki butonunu bul
            sonraki_btn = None
            for btn in next_buttons:
                if "Sonraki" in btn.text:
                    sonraki_btn = btn
                    break
            
            if sonraki_btn and "disabled" not in sonraki_btn.get_attribute("class"):
                print("Sonraki sayfaya geçiliyor...")
                
                # Butona JavaScript ile tıkla
                driver.execute_script("arguments[0].click();", sonraki_btn.find_element(By.TAG_NAME, "a"))
                
                # Yeni sayfanın yüklenmesini bekle
                bekle.until(EC.staleness_of(rows[0]))
                bekle.until(EC.presence_of_element_located((By.CSS_SELECTOR, "table tbody tr")))
                
                sayfa_no += 1
                time.sleep(1.5)
                print(f"{sayfa_no}. sayfaya geçildi")
            else:
                print("Son sayfaya ulaşıldı.")
                sonraki_sayfa_var = False
                
        except Exception as e:
            print(f"Sayfa geçiş hatası: {str(e)}")

            try:
                next_btn = bekle.until(EC.element_to_be_clickable(
                    (By.XPATH, "//a[@class='page-link' and contains(text(), 'Sonraki')]")))
                
                parent_li = next_btn.find_element(By.XPATH, "./..")
                if "disabled" not in parent_li.get_attribute("class"):
                    driver.execute_script("arguments[0].click();", next_btn)
                    sayfa_no += 1
                    print(f"{sayfa_no}. sayfaya geçildi")
                    time.sleep(2)
                else:
                    print("Son sayfaya ulaşıldı.")
                    sonraki_sayfa_var = False
            except:
                print("Sayfa geçişi başarısız, işlem sonlandırılıyor.")
                sonraki_sayfa_var = False

    # Verileri Excel'e kaydet
    if veri:
        tablo = pd.DataFrame(veri, columns=["Başlık", "Karar No", "Tarih", "Sayısı", "Mükerrer", "Mevzuat Türü", "Link"])
        # Boş satırları kaldır
        tablo = tablo.dropna(how='all')
        excel_dosya_adi = "resmi_gazete_son_1ay.xlsx"
        tablo.to_excel(excel_dosya_adi, index=False)
        print(f"\n{'='*50}")
        print(f"Toplam {len(veri)} kayıt, {sayfa_no} sayfa başarıyla kaydedildi!")
        print(f"Dosya adı: {excel_dosya_adi}")
    else:
        print("\nKaydedilecek veri bulunamadı")

finally:
    driver.quit()
'''
