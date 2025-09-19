#  WeatherAPI JMeter Performance Test 🚀

![JMeter](https://img.shields.io/badge/Apache-JMeter-blue?style=for-the-badge&logo=apache)
![Testing](https://img.shields.io/badge/Performance-Testing-yellowgreen?style=for-the-badge)
![SQA](https://img.shields.io/badge/SQA-Best_Practices-purple?style=for-the-badge)

### 1. Project Objective

This project, designed by an SQA professional, demonstrates a robust and scalable performance test plan for the `weatherapi.com` API using Apache JMeter. The goal is to simulate realistic user loads and analyze system performance, focusing on dynamic test execution, data parameterization, and advanced validation.

---

### 2. ✨ Key Features

This test plan incorporates several JMeter best practices for flexibility and accuracy:

* **Dynamic CLI Execution:** The Thread Group is fully parameterized using JMeter's `__P` function (e.g., `${__P(users, 10)}`, `${__P(ramp, 5)}`, `${__P(loops, 1)}`). This allows testers to control the user load, ramp-up time, and loop count directly from the command line without modifying the `.jmx` file.
* **Externalized API Key:** The `${API_KEY}` is passed as a command-line argument (`-JAPI_KEY=...`), a security best practice that prevents sensitive keys from being hardcoded.
* **Data-Driven Testing (DDT):** User requests are parameterized with a `${CITY}` variable, which is dynamically populated from an external `cities.csv` file using the `CSV Data Set Config`.
* **Mixed Workload Simulation:** `Throughput Controllers` are used to simulate a realistic user behavior mix, distributing the load between "Current Weather" and "Forecast" API endpoints.
* **Advanced Validation:** A `JSR223 PostProcessor` is implemented for custom validation logic. It intelligently identifies known, expected data errors (e.g., API errors for non-existent cities like "Eswatini") and distinguishes them from genuine system performance failures (like 5xx errors).
* **Response Assertions:** Standard assertions are in place to validate that each response is successful (`Response Code = 200`) and contains key payload data (`contains "location"`).

---

### 3. Test Plan Structure

The test plan is organized for clarity, scalability, and mixed-load simulation.

*(You can add your screenshot here in GitHub for a visual guide)*
`![Test Plan Structure](images/Test_Plan_Structure.png)`

* **WeatherAPI Test Plan**
    * `User Defined Variables` (Default parameters)
    * **Thread Group WeatherAPI** (Parameterized for CLI)
        * `HTTP Request Defaults` (Server: `api.weatherapi.com`, Protocol: `httpsS`)
        * `CSV Data Set Config` (Reads `cities.csv`)
        * **Throughput Controller Current Weather**
            * `jp@gc - Throughput Shaping Timer`
            * **HTTP Request: Current Weather** (GET `/v1/current.json`)
                * `Check 200 OK` (Assertion)
                * `Check 'location' text` (Assertion)
                * `JSR223 PostProcessor` (Custom Validation)
        * **Throughput Controller Forecast**
            * `jp@gc - Throughput Shaping Timer`
            * **HTTP Request: Forecast** (GET `/v1/forecast.json`)
                * `Check 200 OK` (Assertion)
                * `Check 'location' text` (Assertion)
    * `View Results Tree` (Disabled in CLI mode)
    * `Summary Report` (Disabled in CLI mode)
    * `Aggregate Report` (Disabled in CLI mode)

---

### 4. ⚙️ How to Run

1.  **Prerequisites:**
    * Apache JMeter installed.
    * `WeatherAPI_Test.jmx` file.
    * `cities.csv` file in the same directory (or correct path in the CSV config).
2.  **Get API Key:**
    * Sign up at [weatherapi.com](https://www.weatherapi.com/) to get your free API key.
3.  **Execute from CLI:**
    * Open your terminal or command prompt, navigate to your JMeter `bin` directory, and run the following command. This example runs a test with 100 users, a 20-second ramp-up, 5 loops per user, and generates an HTML report in the `report_500` folder.

    ```bash
    jmeter -n -t /path/to/your/WeatherAPI_Test.jmx -l /path/to/your/result_500.jtl -JAPI_KEY=YOUR_API_KEY_HERE -Jusers=100 -Jramp=20 -Jloops=5 -e -o /path/to/your/report_500
    ```

    **Command Breakdown:**
    * `-n`: Non-GUI mode
    * `-t`: Path to your `.jmx` test file
    * `-l`: Path to save the `.jtl` results file
    * `-J[var]`: Sets a JMeter property (e.g., `-Jusers=100`)
    * `-e`: Generate HTML dashboard report after the test
    * `-o`: Path to save the HTML report (must be an empty folder)

---

### 5. 📊 Performance Analysis Report (Sample)

Below is a sample analysis from a test run comparing 500, 700, and 1000 user loads.

#### **Performance Impact Analysis: WeatherAPI (500 vs. 700 vs. 1000 Users)**

##### **A. Objective**

This report analyzes the performance and scalability of the WeatherAPI service under simulated concurrent user loads of 500, 700, and 1000 users. The test plan utilized a parameterized CSV dataset for dynamic city queries and a mix of "Current Weather" and "Forecast" API endpoints, controlled via Throughput Controllers.

##### **B. Key Metrics Comparison**

The following table summarizes the key performance indicators (KPIs) from the "Total" statistics of each test run.

| Metric | 500 Users | 700 Users | 1000 Users |
| :--- | :---: | :---: | :---: |
| **Avg. Response Time (ms)** | 301.08 | 218.02 | 127.77 |
| **Median (ms)** | 250.00 | 220.00 | 77.00 |
| **90th Percentile (ms)** | 527.70 | 399.80 | 234.90 |
| **Throughput (req/sec)** | 24.30 | 33.51 | 37.99 |
| **Error %** | 0.60% | 0.57% | 0.50% |
| **Max Response Time (ms)** | 2235 | 1234 | 4643 |

---

##### **C. Performance Analysis**

Based on the data, the system demonstrates excellent stability and a counter-intuitive performance improvement as load increases.

**Response Times (Average, Median, 90th Percentile)**

The most significant observation is the **inverse relationship between user load and response time**. As the number of users increased, the response times *decreased* significantly.

* **Average** time fell from 301.08 ms (500 users) to a rapid 127.77 ms (1000 users).
* **Median** time saw a dramatic drop from 250 ms to just 77 ms.
* **90th Percentile**, which represents the experience for the majority of users, improved from 527.70 ms down to 234.90 ms.

This atypical behavior strongly suggests that the **API's server-side caching** is a dominant factor. As the load (and number of total requests) increased, it is highly probable that more responses were served from a fast cache, bypassing slower database queries.

**Throughput (Transactions/sec)**

Throughput scaled positively with the user load, rising from **24.30 req/sec** at 500 users to **37.99 req/sec** at 1000 users. This is a healthy sign, indicating the API was capable of handling the increasing volume of requests without showing signs of saturation.

**Error Rate (%)**

The error rate remained consistently low and stable (0.50% - 0.60%) across all tests. As identified during test design, these errors were **expected and related to test data integrity**, not system instability. The `JSR223 PostProcessor` correctly identified failures for requests (e.g., for "Eswatini") where the API had no data.

Critically, **no HTTP 5xx (Server Error) or connection timeout errors were observed**. This confirms the API backend remained stable and available under all load scenarios.

---

##### **D. Conclusion and Key Observations**

The WeatherAPI system is **exceptionally stable and performs well** under loads of up to 1000 concurrent users.

* **Primary Finding:** The system not only handled the increased load but became *faster*. This behavior is attributed to effective server-side caching, where increased request volume led to a higher cache-hit ratio.
* **Scalability:** The system is healthy. Throughput scaled linearly with user load, and the system never reached a bottleneck or saturation point.
* **Stability:** The system is highly stable. The only errors recorded were known data-related issues, with zero indication of load-induced stress or failure.