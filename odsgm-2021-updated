from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.options import Options

from bs4 import BeautifulSoup
import json
import os
import re
from selenium.webdriver.support.select import Select
from bs4.element import Tag


def r_ws(s):
    while '  ' in s:
        s = s.replace('  ', '')
    while '\n' in s:
        s = s.replace('\n', '')
    return s


url = "https://odsgm.meb.gov.tr/www/kazanim-testleri/kategori/107"
chrome_options = Options()
chrome_options.add_experimental_option("detach", True)
driver = webdriver.Chrome(
    "/Users/ataberkw/Downloads/chromedriver", chrome_options=chrome_options)
driver.set_window_position(0, 0)
driver.get(url)

cs: dict = {}
rawEntryData = []

select = Select(driver.find_element_by_name('haber-listesi_length'))
select.select_by_value('100')

soup = BeautifulSoup(driver.page_source, 'lxml')


def cleanTopicName(name: str):
    for a in range(30):
        name = name.replace(" - " + str(a), "")
    return name.replace("(", "").replace(")", "")


def getClassName(name: str):
    if name.find("Mezun ") != -1:
        return "mezun"
    if name.find("12. Sınıf ") != -1:
        return "12"
    if name.find("11. Sınıf ") != -1:
        return "11"
    if name.find("10. Sınıf ") != -1:
        return "10"
    if name.find("9. Sınıf ") != -1:
        return "9"
    if name.find("8. Sınıf ") != -1:
        return "8"
    if name.find("7. Sınıf ") != -1:
        return "7"
    if name.find("6. Sınıf ") != -1:
        return "6"
    if name.find("5. Sınıf ") != -1:
        return "5"
    return "?"


def getLessonName(name: str):
    name = name.replace(" Kazanım Testleri", "")
    return name.replace("Mezun ", "").replace("12. Sınıf ", "").replace("11. Sınıf ", "").replace("10. Sınıf ", "").replace("9. Sınıf ", "").replace("8. Sınıf ", "").replace("7. Sınıf ", "").replace("6. Sınıf ", "").replace("5. Sınıf ", "")


lessonCount = 0
lessonId: int = 0
topicId: int = 0

allMainEntries: list = soup.findAll('h4')
for mainEntry in allMainEntries:
    mainEntry: Tag
    print(mainEntry)
    mainEntryNameRaw = mainEntry.text

    parentA: Tag = mainEntry.findParent()
    mainEntryUrl = "https://odsgm.meb.gov.tr" + parentA.attrs["href"]

    classKey = getClassName(mainEntryNameRaw)
    lessonName = getLessonName(mainEntryNameRaw)
    if not classKey in cs:
        cs[classKey] = {}
    cs[classKey][lessonName] = {"url": mainEntryUrl, "topics": []}
    rawEntryData.append(
        {"url": mainEntryUrl, "lessonId": lessonId, "lessonName": lessonName, "classKey": classKey})
    lessonId += 1


for a in range(float(len(allMainEntries)/5).__ceil__()):
    lastTopicName: str = ""
    windows: list = driver.window_handles
    driver.switch_to.window(windows[0])
    for b in range(5):
        i: int = a*5 + b
        if i >= len(allMainEntries):
            break
        entry = allMainEntries[i]
        entryUrl = rawEntryData[i]["url"]
        driver.execute_script("window.open('"+entryUrl+"','_blank');")
    for b in range(5):
        i: int = a*5 + b
        if i >= len(allMainEntries):
            break
        windows: list = driver.window_handles
        driver.switch_to.window(windows[len(windows) - 1])
        lsoup = BeautifulSoup(driver.page_source, "lxml")
        ltopics = lsoup.findChildren("p", {"class": "MsoNormal"})
        ed = rawEntryData[i]
        for ptopic in ltopics:
            ptopic: Tag
            atopic: Tag = ptopic.findChild("a")
            topicName: str = cleanTopicName(atopic.text)
            if topicName == "":
                continue
            if not cs[ed["classKey"]][ed["lessonName"]]["topics"].__contains__({"name": topicName, "lessonId": ed["lessonId"]}):
                cs[ed["classKey"]][ed["lessonName"]
                                   ]["topics"].append({"name": topicName, "lessonId": ed["lessonId"]})
            if not "lessonId" in cs[ed["classKey"]][ed["lessonName"]]:
                cs[ed["classKey"]][ed["lessonName"]]["lessonId"] = ed["lessonId"]
        driver.close()

print("\n\n\n " + json.dumps(cs))
