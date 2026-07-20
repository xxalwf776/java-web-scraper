# ScraperAPI Java: Complete Guide to Web Scraping in Java Without Getting Blocked — How to Set Up the SDK, Handle Anti-Bot Sites, and Which Plan Should You Choose? (Includes Full Pricing Breakdown & Working Code Examples)

There's a specific kind of frustration that Java developers know well: you write a clean HTTP request, send it to a website, and get back a 403 error, a blank page, or — worse — a CAPTCHA. The data you want is sitting right there in the browser, but the moment your code touches it, the wall goes up.

That's the problem `scraperapi java` searches are almost always trying to solve. Not "how do I write a loop that fetches URLs" — that part is easy — but "how do I actually get the HTML back reliably, at scale, without building and babysitting my own proxy infrastructure."

This guide covers both halves of the story: how to scrape with Java, and how ScraperAPI plugs into that workflow to handle the infrastructure headaches so you can focus on the data.

---

**Why Java for Web Scraping?**

Python gets all the press for web scraping, and for good reason — it's fast to prototype, the libraries are elegant, and the community is enormous. But Java has a real argument to make, especially if you're already running JVM-based services.

First, Java is genuinely fast at processing large volumes of data. When you're running a pipeline that's processing millions of records or serving a real-time backend, the JVM's throughput advantages matter. Second, if your data pipeline already lives on the JVM, adding a Java scraper is a natural extension — no polyglot infrastructure, no serialization gymnastics. Third, libraries like WebMagic give you distributed, multi-threaded crawlers that would take considerable setup to replicate in Python's single-threaded ecosystem.

The catch is anti-bot protection. Java's standard HTTP libraries — `HttpClient`, Jsoup's `connect()`, HtmlUnit — are trivially fingerprinted by modern bot-detection systems like Cloudflare, Datadome, or PerimeterX. A bare-metal Java request often doesn't look like a real browser at all, and these systems know it immediately.

That's where a scraping API comes in. Instead of building and maintaining proxy rotation, CAPTCHA solving, and browser fingerprint management yourself, you delegate those concerns to a service that's already solved them — and you keep writing clean Java code against a simple HTTP endpoint.

---

**What ScraperAPI Actually Does**

ScraperAPI is a web scraping API that sits between your code and the target website. You send one API call with your target URL; ScraperAPI routes the request through a pool of over 40 million IPs across 50+ countries, handles CAPTCHA solving, manages browser rendering for JavaScript-heavy sites, and returns clean HTML or structured JSON. You get the page. The proxy rotation, retries, and anti-bot bypasses are handled on their side.

For Java developers specifically, there are two integration paths: the official Java SDK (Maven), and direct HTTP calls using Java's `HttpClient` or a library like Jsoup or HtmlUnit. Both are straightforward. The SDK is cleaner for most use cases; the direct HTTP approach works better if you're already deep into a particular library's ecosystem.

👉 [Get started with a free ScraperAPI trial — no credit card required](https://www.scraperapi.com/?fp_ref=coupons)

---

**Method 1: The ScraperAPI Java SDK (Maven)**

ScraperAPI publishes an official SDK to Maven Central. This is the simplest integration path.

**Step 1: Add the dependency to your `pom.xml`**

xml
<dependency>
    <groupId>com.scraperapi</groupId>
    <artifactId>sdk</artifactId>
    <version>1.2</version>
</dependency>


The artifact is at `com.scraperapi:sdk:1.2` on Maven Central. If you're using Gradle, the equivalent is `implementation 'com.scraperapi:sdk:1.2'`.

**Step 2: Basic GET request**

java
package com.example;
import com.scraperapi.ScraperApiClient;

public class Main {
    public static void main(String[] args) {
        ScraperApiClient client = new ScraperApiClient("YOUR_API_KEY");
        String result = client.get("https://example.com/").result();
        System.out.println(result);
    }
}


That's the entire integration for a standard page request. One import, one client initialization, one method call. You get back the full HTML of the target page.

**Step 3: Enable JavaScript rendering**

For SPAs, React pages, or any site that loads content dynamically:

java
ScraperApiClient client = new ScraperApiClient("YOUR_API_KEY");
String result = client.get("https://example.com/")
    .render(true)
    .result();


**Step 4: Geotargeting**

If your scraping needs region-specific data (prices, availability, localized content):

java
ScraperApiClient client = new ScraperApiClient("YOUR_API_KEY");
String result = client.get("https://example.com/")
    .render(true)
    .country_code("us")
    .result();


The `country_code` parameter routes your request through a proxy in the specified country. Available countries depend on your plan tier — more on that in the pricing section.

---

**Method 2: Direct HTTP with Java's Built-in HttpClient**

If you'd rather not add the SDK dependency and prefer raw HTTP, Java 11's `HttpClient` works perfectly with ScraperAPI's endpoint pattern:

java
import java.net.URI;
import java.net.URLEncoder;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.charset.StandardCharsets;

public class ScraperApiDirect {
    public static void main(String[] args) throws Exception {
        String apiKey = "YOUR_API_KEY";
        String targetUrl = URLEncoder.encode("https://example.com/", StandardCharsets.UTF_8);
        String endpoint = "http://api.scraperapi.com?api_key=" + apiKey + "&url=" + targetUrl;

        HttpClient client = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(endpoint))
            .GET()
            .build();

        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        System.out.println(response.body());
    }
}


To enable JavaScript rendering, append `&render=true` to the endpoint string. To add geotargeting, append `&country_code=us`. The API accepts all parameters as query string values, so the pattern scales to any combination of options.

---

**Method 3: ScraperAPI with HtmlUnit**

HtmlUnit is a headless browser library for Java — useful when you need to simulate browser interactions or parse JavaScript-heavy pages with a browser engine rather than a pure HTTP call. Integrating it with ScraperAPI is straightforward:

java
import com.gargoylesoftware.htmlunit.BrowserVersion;
import com.gargoylesoftware.htmlunit.WebClient;
import com.gargoylesoftware.htmlunit.html.DomNode;
import com.gargoylesoftware.htmlunit.html.DomNodeList;
import com.gargoylesoftware.htmlunit.html.HtmlPage;

public class ScraperApiHtmlUnit {
    public static void main(String[] args) throws Exception {
        String apiKey = "YOUR_API_KEY";
        String targetUrl = "https://quotes.toscrape.com";
        String scraperApiUrl = "http://api.scraperapi.com?api_key=" + apiKey + "&url=" + targetUrl;

        WebClient webClient = new WebClient(BrowserVersion.CHROME);
        webClient.getOptions().setUseInsecureSSL(true);
        webClient.getOptions().setCssEnabled(false);
        webClient.getOptions().setJavaScriptEnabled(false);
        webClient.getOptions().setThrowExceptionOnFailingStatusCode(false);
        webClient.getOptions().setThrowExceptionOnScriptError(false);

        HtmlPage page = (HtmlPage) webClient.getPage(scraperApiUrl);
        DomNodeList<DomNode> quoteBlocks = page.querySelectorAll(".quote");

        for (DomNode quote : quoteBlocks) {
            String text = quote.querySelector(".text").asNormalizedText();
            String author = quote.querySelector(".author").asNormalizedText();
            System.out.println(text + " — " + author);
        }

        webClient.close();
    }
}


One important note for HtmlUnit specifically: **don't use ScraperAPI in proxy mode with HtmlUnit**. ScraperAPI uses query string authentication, which doesn't align with HtmlUnit's proxy model. Always use the API endpoint approach as shown above.

The `pom.xml` dependencies for this approach:

xml
<dependency>
    <groupId>net.sourceforge.htmlunit</groupId>
    <artifactId>htmlunit</artifactId>
    <version>2.70.0</version>
</dependency>
<dependency>
    <groupId>io.github.cdimascio</groupId>
    <artifactId>java-dotenv</artifactId>
    <version>5.2.2</version>
</dependency>


---

**Method 4: ScraperAPI + Jsoup (Best of Both Worlds)**

Jsoup is still the gold standard Java library for HTML parsing — its CSS selector API is clean and expressive. The combination that works best in practice is: ScraperAPI handles the fetch (bypassing blocks, proxies, CAPTCHAs), and Jsoup handles the parsing (extracting structured data from the returned HTML).

java
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.select.Elements;

import java.net.HttpURLConnection;
import java.net.URL;
import java.net.URLEncoder;
import java.io.BufferedReader;
import java.io.InputStreamReader;

public class ScraperApiJsoup {
    public static void main(String[] args) throws Exception {
        String apiKey = "YOUR_API_KEY";
        String targetUrl = URLEncoder.encode("https://quotes.toscrape.com", "UTF-8");
        String requestUrl = "http://api.scraperapi.com?api_key=" + apiKey + "&url=" + targetUrl;

        URL url = new URL(requestUrl);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        conn.setRequestMethod("GET");

        BufferedReader in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
        StringBuilder rawHtml = new StringBuilder();
        String line;
        while ((line = in.readLine()) != null) rawHtml.append(line);
        in.close();

        // Hand off the HTML to Jsoup for parsing
        Document doc = Jsoup.parse(rawHtml.toString());
        Elements quotes = doc.select(".quote .text");
        quotes.forEach(q -> System.out.println(q.text()));
    }
}


This pattern separates concerns cleanly: ScraperAPI owns the network layer (with all its complexity), Jsoup owns the parsing layer (with its elegance). It's the approach most production Java scrapers end up at.

---

**Understanding the Credit System Before You Choose a Plan**

This is the part that trips up most people, and it's worth being direct about it.

Every request you send through ScraperAPI costs API credits — but not all requests cost the same. The base rate is **1 credit per standard page request**. The actual cost multiplies based on the domain and parameters you use:

| Domain / Feature | Credit Cost |
|---|---|
| Standard page (flat rate) | 1 credit |
| Amazon | 5 credits |
| Google / Bing (all subdomains) | 25 credits |
| LinkedIn | 30 credits |
| Cloudflare bypass | +10 credits |
| Datadome bypass | +10 credits |
| PerimeterX / Human bypass | +10 credits |
| `render=true` (JS rendering) | +10 credits |
| `premium=true` | +10 credits |
| `screenshot=true` | +10 credits |
| `ultra_premium=true` | +30 credits |
| `premium=true` + `render=true` | 25 credits total |
| `ultra_premium=true` + `render=true` | 75 credits total |

What this means in practice: the "100,000 credits" on the entry plan sounds like a lot. Against a plain blog or news site with no anti-bot protection, it's exactly that — 100,000 pages. Against Amazon with JS rendering enabled, that's 5 credits for the domain plus 10 for render = 15 credits per request, which means roughly **6,600 pages** from the same plan. Before choosing a plan, run a few test requests through ScraperAPI's [Domain Multiplier tool](https://dashboard.scraperapi.com/home/domain-multiplier) to see your real per-request cost.

The one genuinely fair detail: **you only pay for successful requests** (HTTP 200 or 404 responses). Failed scrapes don't consume credits.

---

**Full ScraperAPI Plans Comparison**

All plans include: JavaScript rendering, residential proxy rotation, JSON auto-parsing, custom headers, CAPTCHA / anti-bot bypass, custom sessions, automatic retries, unlimited bandwidth, and a 99.9% uptime guarantee. The differences are volume, concurrency limits, and geotargeting scope.

| Plan | Monthly Price | Annual Price (10% off) | API Credits/Month | Concurrent Threads | Geotargeting | Get Started |
|---|---|---|---|---|---|---|
| **Free** | $0 | — | 1,000 (ongoing) + 5,000 trial | 5 | — |  [Start Free](https://www.scraperapi.com/?fp_ref=coupons) |
| **Hobby** | $49/mo | $44.10/mo | 100,000 | 20 | US & EU only |  [Get Hobby](https://www.scraperapi.com/?fp_ref=coupons) |
| **Startup** | $149/mo | $134.10/mo | 1,000,000 | 50 | US & EU only |  [Get Startup](https://www.scraperapi.com/?fp_ref=coupons) |
| **Business** | $299/mo | $269.10/mo | 3,000,000 | 100 | Global |  [Get Business](https://www.scraperapi.com/?fp_ref=coupons) |
| **Scaling** ⭐ Most Popular | $475/mo | $427.50/mo | 5,000,000 | 200 | Global |  [Get Scaling](https://www.scraperapi.com/?fp_ref=coupons) |
| **Professional** | $975/mo | $877.50/mo | 10,500,000 | 300 | Global |  [Get Professional](https://www.scraperapi.com/?fp_ref=coupons) |
| **Advanced** | $1,975/mo | $1,777.50/mo | 21,500,000 | 500 | Global |  [Get Advanced](https://www.scraperapi.com/?fp_ref=coupons) |
| **Enterprise** | Custom | Custom | 22,000,000+ | 500+ | Global |  [Contact Sales](https://www.scraperapi.com/?fp_ref=coupons) |

A few things the table doesn't surface on its own:

- **Geotargeting is gated by tier.** Hobby and Startup are locked to US & EU proxies only. If you need to scrape region-specific data from Asia, Latin America, or other regions, you need Business or higher.
- **Pay-as-you-go (PAYG) starts at the Scaling tier.** Hobby, Startup, and Business plans hard-cap you when credits run out — no PAYG overflow. Scaling, Professional, Advanced, and Enterprise all include PAYG so you're never cut off mid-month.
- **Credits don't roll over.** Unused credits reset at renewal, so right-sizing your plan to actual monthly consumption matters more than overshooting "just in case."
- **Analytics history** is capped at 30 days for Hobby and Startup; Business and above get unlimited history.
- **Annual billing saves 10%** automatically at checkout — no coupon code needed.

---

**Which Plan Should You Actually Pick?**

This is the real question, and the answer depends almost entirely on what you're scraping.

**Hobby ($49/mo)** makes sense if you're building a personal project, a proof of concept, or a side tool. Checking competitor prices on a handful of products, monitoring a few dozen pages, prototyping a Java scraper before pitching it internally — that's the Hobby tier. For plain unprotected pages, 100,000 credits goes a long way. Just run your target URLs through the cost estimator first.

**Startup ($149/mo)** is the right step for a small SaaS product, an agency running scraping jobs for a handful of clients, or a developer who's graduated from "testing it out" to "this is actually in production." The jump to 1,000,000 credits and 50 concurrent threads is substantial, though you're still limited to US/EU geotargeting.

**Business ($299/mo)** unlocks two things the lower tiers don't offer: global geotargeting and unlimited analytics history. If your scraping needs to reach outside the US and EU, or if your production system depends on dashboard data beyond 30 days, this is the floor you need.

**Scaling and above ($475/mo+)** is where PAYG overflow becomes available. If you're running production infrastructure with unpredictable traffic spikes — scraping triggered by user events, for instance — the ability to continue past your credit limit at a fixed rate is worth more than the raw credit volume increase.

---

**Practical Tips for Java + ScraperAPI in Production**

A few things that matter once you're past the "hello world" stage:

**Always store your API key in environment variables, not hardcoded in source.** The SDK guide shows `new ScraperApiClient("API_KEY")` for simplicity, but in any real codebase you want `System.getenv("SCRAPERAPI_KEY")` or a dotenv file excluded from version control.

**Use the Domain Multiplier before choosing a plan.** The dashboard's cost estimator lets you paste any URL and see the exact credit cost per request for that domain with your chosen parameters. Run your top-10 target URLs through it before committing to a tier.

**Disable CSS in HtmlUnit when you don't need styling.** It speeds up requests significantly: `webClient.getOptions().setCssEnabled(false)`. If you're not rendering visual output, there's no reason to process stylesheets.

**Let ScraperAPI handle rendering for JS-heavy pages, not HtmlUnit's JavaScript engine.** HtmlUnit's JS support breaks frequently on modern frameworks. Use `render=true` in your ScraperAPI request instead — it routes the request through a full headless browser on ScraperAPI's side.

**Handle rate limiting gracefully.** Even with ScraperAPI managing the proxy rotation, your own request loop should include reasonable delays and retry logic. A burst of 500 concurrent requests where your plan supports 20 threads will create a queue backlog — scale your concurrency to match your plan's thread limit.

**The 7-day free trial is genuinely useful for testing.** New accounts get 5,000 credits on signup, no credit card required. Use that trial against your real target URLs — not a toy example — and watch the credit consumption in the dashboard. That number tells you which plan actually fits your workload before you spend anything.

---

**What Users Actually Say**

ScraperAPI sits at 4.5/5 on Trustpilot and 4.4/5 on G2, with the pattern across reviews being remarkably consistent. The praise focuses on three things: the documentation is clean and practically oriented, integration is fast (most developers describe getting up and running in under an hour), and the support team is responsive when things go sideways.

The criticism is also consistent: the credit math is less transparent than the headline numbers suggest, particularly once you start layering domain multipliers and rendering costs. Several independent benchmarks note that ScraperAPI performs very well on mainstream targets — Amazon, Google, standard e-commerce — but can be less consistent on sites with frequently-changing, aggressive anti-bot systems.

For the Java use case specifically, the combination of the official Maven SDK and a standard Jsoup parsing layer is a well-tested pattern. It handles the "how do I get the HTML" problem cleanly, which is the part of Java web scraping that's genuinely difficult to solve yourself at any meaningful scale.

---

**Frequently Asked Questions**

**Does the Java SDK support async requests?**
The current SDK (`com.scraperapi:sdk:1.2`) uses synchronous calls. For async patterns in Java, use the direct HTTP approach with `CompletableFuture` and `HttpClient`'s `sendAsync()` method against the ScraperAPI endpoint.

**What's the difference between `premium=true` and `ultra_premium=true`?**
Both activate higher-quality proxy pools for harder targets. `ultra_premium` provides access to the highest-quality residential and mobile IPs for the most aggressive bot-detection environments — at 30 additional credits per request (75 with rendering). It's only available on paid plans.

**Can I use `country_code` on the Hobby plan?**
The `country_code` parameter is available on all plans, but Hobby and Startup are limited to US and EU proxy pools. Specifying a country outside that range on those tiers won't work as expected.

**What happens if I run out of credits mid-month on the Hobby plan?**
Hobby, Startup, and Business plans hard-cap when credits are exhausted. You'd need to upgrade to the next tier or contact support. Scaling, Professional, Advanced, and Enterprise plans include PAYG overflow so you can keep running at a fixed additional rate.

**Is there a refund policy?**
Yes — ScraperAPI offers a 7-day no-questions-asked refund if you're not satisfied with the service.

---

Ready to stop fighting anti-bot systems and start actually scraping? The free trial gives you 5,000 credits against your real targets — no credit card needed — which is enough to run a meaningful test of the Jsoup or HtmlUnit integration patterns above and see exactly where you'll land on the credit consumption curve.

👉 [Start your free ScraperAPI trial and test it against your actual Java scraping targets](https://www.scraperapi.com/?fp_ref=coupons)
