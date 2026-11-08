<div align="center">

# 🚌 RedBus Automation · Selenium WebDriver Project

**Java + Selenium WebDriver 4 script that automates a bus search on redbus.in, applies filters, handles lazy loading and lists every bus found**

<p>
  <img src="https://img.shields.io/badge/Java-11%2B-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white" alt="Java 11+"/>
  <img src="https://img.shields.io/badge/Selenium-4.44.0-43B02A?style=for-the-badge&logo=selenium&logoColor=white" alt="Selenium 4.44.0"/>
  <img src="https://img.shields.io/badge/Maven-Build-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white" alt="Maven"/>
  <img src="https://img.shields.io/badge/Chrome-WebDriver-4285F4?style=for-the-badge&logo=googlechrome&logoColor=white" alt="Chrome WebDriver"/>
</p>
<p>
  <img src="https://img.shields.io/badge/Eclipse-IDE-2C2255?style=for-the-badge&logo=eclipseide&logoColor=white" alt="Eclipse IDE"/>
  <img src="https://img.shields.io/badge/XPath-Locators-6A1B9A?style=for-the-badge" alt="XPath locators"/>
  <img src="https://img.shields.io/badge/Explicit-Waits-00897B?style=for-the-badge" alt="Explicit waits"/>
  <img src="https://img.shields.io/badge/Target-redbus.in-D84E55?style=for-the-badge" alt="Target redbus.in"/>
</p>
<p>
  <img src="https://img.shields.io/badge/Built%20with-Claude%20Code-D97757?style=for-the-badge" alt="Built with Claude Code"/>
</p>

[Overview](#overview) · [Workflow](#workflow) · [Tech stack](#tech-stack) · [Quick start](#quick-start) · [Key concepts](#key-concepts) · [Challenges](#challenges) · [Improvements](#improvements) · [Author](#author)

</div>

---

<a id="overview"></a>
## 📖 Overview

A Selenium WebDriver (Java) script that automates a bus search on [redbus.in](https://www.redbus.in). It searches for buses from **Mumbai** to **Pune**, applies the **Primo** and **Evening** filters, scrolls through the lazily loaded results until the whole list is on the page, and then prints every bus operator's name along with the total count.

<a id="workflow"></a>
## 🔄 What the script does

1. Opens Chrome (maximized) and goes to `https://www.redbus.in`.
2. Clicks the source/destination widget to open the location search.
3. Types **Mumbai** as the source and picks the exact match from the suggestions.
4. Types **Pune** as the destination and picks the exact match.
5. Clicks **Search**.
6. Applies the **Primo** filter, then the **Evening** departure-time filter.
7. Waits for the results subtitle (e.g. "_N buses_") to update and prints it.
8. Keeps scrolling the results until the **"End of list"** marker shows up, so every result is loaded.
9. Prints the name of each bus operator and the total number of buses found.

<a id="tech-stack"></a>
## 🛠️ Tech stack

| Tool | Version / Notes |
|------|-----------------|
| Java | JDK 11 or later (Selenium 4 needs at least Java 11) |
| Selenium Java | 4.44.0 |
| Build tool | Maven |
| Browser | Google Chrome (Selenium Manager downloads the matching ChromeDriver automatically) |
| IDE | Eclipse (project files included) |

## 📁 Project structure

```
RedBus-Automation/
├── pom.xml                                   # Maven config with the Selenium dependency
└── src/main/java/com/redbus/
    ├── RedBusApplicationAutomation.java      # The automation script (main method)
    └── package-info.java
```

<a id="quick-start"></a>
## 🚀 Quick start

### Prerequisites
- JDK 11+
- Maven 3.x
- Google Chrome

### Clone
```bash
git clone https://github.com/ArpitChoubey/RedBus-Automation.git
cd RedBus-Automation
```

### Run

**From Eclipse / IntelliJ:** import it as a Maven project, open `RedBusApplicationAutomation.java` and run it as a Java Application.

**From the command line:**
```bash
mvn compile exec:java -Dexec.mainClass="com.redbus.RedBusApplicationAutomation"
```

### 🖥️ Sample output
```
<N> buses
<Bus operator 1>
<Bus operator 2>
...
Total Number of Buses Loaded with Primo and Evening Filter<N>
```

<a id="key-concepts"></a>
## 🧠 Key concepts used

- **Explicit waits** (`WebDriverWait` + `ExpectedConditions`): `visibilityOfElementLocated`, `elementToBeClickable`, `visibilityOfAllElementsLocatedBy`, `numberOfElementsToBeMoreThan` and `textToBePresentInElementLocated`
- **Partial-match XPath** with `contains(@class, ...)` and `contains(text(), ...)`
- **Relative (chained) locators** such as `.//div[...]`, which search only inside a parent element like a result row
- **Active element handling** with `driver.switchTo().activeElement()`
- **JavascriptExecutor** for scrolling with `scrollIntoView`
- A **reusable helper method** (`selectLocation`) for picking both source and destination

<a id="challenges"></a>
## 🧩 Challenges faced and how they were solved

### 1. Dynamic, auto-generated class names
RedBus is a modern React-style app. Its class names often carry generated suffixes (for example `srcDestWrapper___xyz12`) that can change between builds, so locators that match the full class name break quickly.
**Solution:** match only the stable part of the class with XPath `contains()`, as in `//div[contains(@class,"srcDestWrapper")]`, `//li[contains(@class,"tupleWrapper")]` and `//button[contains(@class,"searchButtonWrapper")]`.

### 2. Finding the search input
The source/destination field isn't a normal input you can click directly. Clicking the wrapper opens a search panel, and an input inside it gets focus automatically. That input has no reliable locator of its own.
**Solution:** use `driver.switchTo().activeElement()` to grab whichever element has focus and type into it. The same approach works for the destination step, because the site moves focus there after the source is chosen.

### 3. Picking the right location from the suggestions
Typing a city name returns suggestions grouped into several categories (the city itself, bus stops inside it, nearby places, and so on), and they load asynchronously. Clicking too early, or clicking the first item you see, can select the wrong place.
**Solution:** wait until there are more than two suggestion categories (`numberOfElementsToBeMoreThan`), take the first category, and loop through its entries until one matches the city name exactly with `equalsIgnoreCase`.

### 4. Filters update the page asynchronously
After **Primo** and **Evening** are clicked, the results list re-renders in the background. Reading the results right away would return stale data from before the filter.
**Solution:** wait for each filter to be clickable before clicking it, then wait for the results subtitle to contain the text `"buses"` before reading it, so the page has finished updating.

### 5. Lazy loading / infinite scroll
The results page doesn't render every bus at once. More rows load only as you scroll down, so a single `findElements` call returns just the first batch.
**Solution:** a scroll loop that:
- collects the rows currently visible,
- checks whether the **"End of list"** marker has appeared and stops if it has,
- otherwise uses `JavascriptExecutor` to scroll the **third-to-last** row into view. Scrolling slightly before the very last row reliably triggers the next batch to load.

### 6. Reading data from each result row
Each result card holds many nested elements. A global XPath for the operator name would return names from every row, or from the wrong row.
**Solution:** find the row container (`tupleWrapper`) first, then use a **relative XPath** (`.//div[contains(@class,"travelsName")]`) so each lookup stays inside its own row.

### 7. Timing and flakiness
Network speed and animations (such as smooth scrolling) make fixed `Thread.sleep()` delays unreliable: too short and the test fails, too long and it wastes time.
**Solution:** use explicit waits only, with a 30-second `WebDriverWait`, so each step continues as soon as the page is ready.

<a id="improvements"></a>
## 📈 Possible improvements

- Close the browser at the end with `driver.quit()`, ideally in a `finally` block
- Move the source, destination and filters into a config file or command-line arguments
- Restructure the code with the **Page Object Model** (POM)
- Add **TestNG/JUnit** with assertions, plus reporting (e.g. Extent Reports)
- Add a maximum number of scroll attempts so the lazy-load loop can't run forever if the "End of list" marker never appears
- Support headless runs for CI (e.g. GitHub Actions)
- Add logging (Log4j/SLF4J) in place of `System.out.println`

<a id="author"></a>
## 👤 Author

**Arpit Choubey** – [GitHub](https://github.com/ArpitChoubey)

> This project is for learning and practice only. It is not affiliated with redBus.
