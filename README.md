# pdf-download

To resolve the issue of downloading PDFs using C# Selenium in headless mode without encountering the "Save As" popup, follow these steps based on the latest practices and search results:

---

### **Root Cause Analysis**
The "Save As" popup in headless Chrome typically appears because the browser lacks a default download path configuration or due to missing Chrome DevTools Protocol (CDP) settings. This problem often arises when ChromeDriver or Chrome versions are outdated, or the CDP command for downloads is not properly configured.

---

### **Solution: Configure Headless Chrome for Direct PDF Downloads**
#### **1. Update ChromeDriver and Chrome**
Ensure you are using **ChromeDriver v77.0.3865.40 or newer** and **Chrome v96+**, as older versions lack proper headless download support.

#### **2. Configure ChromeOptions in C#**
Set up headless mode, disable automation flags, and define a default download directory:
```csharp
using OpenQA.Selenium.Chrome;

var options = new ChromeOptions();
options.AddArgument("--headless=new"); // Use unified headless mode
options.AddArgument("--window-size=1920,1080");
options.AddArgument("--disable-extensions");
options.AddArgument("--disable-popup-blocking");
options.AddArgument("--disable-gpu");
options.AddUserProfilePreference("download.default_directory", "C:\\Downloads"); // Set download path
options.AddUserProfilePreference("download.prompt_for_download", false); // Disable prompt
options.AddExcludedArgument("enable-automation"); // Bypass detection
var driver = new ChromeDriver(options);
```

#### **3. Use Chrome DevTools to Set Download Behavior**
Execute the CDP command `Page.setDownloadBehavior` to override default settings:
```csharp
var devTools = driver.GetDevTools();
await devTools.CreateSession();
var parameters = new Dictionary<string, object>
{
    { "behavior", "allow" },
    { "downloadPath", "C:\\Downloads" }
};
await devTools.SendCommand("Page.setDownloadBehavior", parameters);
```
This command explicitly allows downloads and defines the save directory.

#### **4. Trigger the Download**
Click the download button and wait for the file to save:
```csharp
driver.Navigate().GoToUrl("https://example.com/pdf-download");
var downloadButton = driver.FindElement(By.Id("download-button"));
downloadButton.Click();

// Wait for the file to download (adjust timeout as needed)
var wait = new WebDriverWait(driver, TimeSpan.FromSeconds(30));
wait.Until(d => new DirectoryInfo("C:\\Downloads").GetFiles("*.pdf").Length > 0);
```

---

### **Alternative Approaches**
#### **1. Use Non-Headless Mode with Hidden UI**
If headless mode still fails, run Chrome in the background without visible UI:
```csharp
options.AddArgument("--start-maximized");
options.AddArgument("--window-position=-32000,-32000");
```

#### **2. Direct PDF URL Fetch**
Extract the PDF URL via Selenium and download it using `HttpClient`:
```csharp
var pdfUrl = driver.FindElement(By.Id("pdf-link")).GetAttribute("href");
using (var httpClient = new HttpClient())
{
    var pdfBytes = await httpClient.GetByteArrayAsync(pdfUrl);
    File.WriteAllBytes("C:\\Downloads\\file.pdf", pdfBytes);
}
```

#### **3. Browser Profile Configuration**
Use a preconfigured Chrome profile with download settings:
```csharp
options.AddArgument("--user-data-dir=C:\\ChromeProfile");
```

---

### **Common Pitfalls & Fixes**
- **Outdated ChromeDriver**: Update to the latest version from [ChromeDriver Downloads](https://chromedriver.chromium.org/).
- **Permission Issues**: Ensure the download directory is writable.
- **Incorrect CDP Command Syntax**: Verify the `Page.setDownloadBehavior` parameters.
- **Ad-Blocking Extensions**: Disable them using `options.AddArgument("--disable-extensions")`.

---

### **Conclusion**
The primary solution involves configuring ChromeOptions and using the `Page.setDownloadBehavior` CDP command. If issues persist, consider alternative methods like direct HTTP downloads. For further debugging, enable Chrome logs with `options.SetLoggingPreference(LogType.Driver, LogLevel.All)`.
