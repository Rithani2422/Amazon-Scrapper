# Amazon-Scrapper
// Import necessary libraries
import java.io.*;
import java.util.*;
import org.openqa.selenium.*;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;

// Class to represent a product
class Product {
    private String name;
    private String price;
    private String rating;
    private String reviews;
    private String availability;
    private String imageUrl;

    // Constructor
    public Product(String name, String price, String rating, String reviews, String availability, String imageUrl) {
        this.name = name;
        this.price = price;
        this.rating = rating;
        this.reviews = reviews;
        this.availability = availability;
        this.imageUrl = imageUrl;
    }

    // Getters
    public String getName() { return name; }
    public String getPrice() { return price; }
    public String getRating() { return rating; }
    public String getReviews() { return reviews; }
    public String getAvailability() { return availability; }
    public String getImageUrl() { return imageUrl; }

    @Override
    public String toString() {
        return "Product [Name=" + name + ", Price=" + price + ", Rating=" + rating + ", Reviews=" + reviews + ", Availability=" + availability + ", ImageUrl=" + imageUrl + "]";
    }
}

// Main scraper class
class AmazonScraper {
    private WebDriver driver;

    // Constructor
    public AmazonScraper() {
        // Set up WebDriver (assumes ChromeDriver is in PATH)
        System.setProperty("webdriver.chrome.driver", "path/to/chromedriver");
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--headless");
        this.driver = new ChromeDriver(options);
    }

    // Method to search for products on Amazon
    public List<Product> searchProducts(String searchQuery, int maxPages) {
        List<Product> products = new ArrayList<>();
        try {
            String baseUrl = "https://www.amazon.com/s?k=" + searchQuery.replace(" ", "+");
            for (int page = 1; page <= maxPages; page++) {
                driver.get(baseUrl + "&page=" + page);

                // Wait for the page to load
                Thread.sleep(2000);

                // Extract product elements
                List<WebElement> productElements = driver.findElements(By.cssSelector(".s-main-slot .s-result-item"));
                for (WebElement productElement : productElements) {
                    try {
                        String name = productElement.findElement(By.cssSelector("h2 a span")).getText();
                        String price = "N/A";
                        try {
                            price = productElement.findElement(By.cssSelector(".a-price .a-offscreen")).getText();
                        } catch (NoSuchElementException ignored) {}
                        String rating = "N/A";
                        try {
                            rating = productElement.findElement(By.cssSelector(".a-icon-alt")).getAttribute("innerHTML");
                        } catch (NoSuchElementException ignored) {}
                        String reviews = "N/A";
                        try {
                            reviews = productElement.findElement(By.cssSelector(".s-link-style span.a-size-base")).getText();
                        } catch (NoSuchElementException ignored) {}
                        String availability = productElement.findElement(By.cssSelector(".s-availability span")).getText();
                        String imageUrl = productElement.findElement(By.cssSelector(".s-image")).getAttribute("src");

                        products.add(new Product(name, price, rating, reviews, availability, imageUrl));
                    } catch (NoSuchElementException ignored) {
                        // Skip products with missing fields
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return products;
    }

    // Method to save products to a CSV file
    public void saveToCSV(List<Product> products, String fileName) {
        try (PrintWriter writer = new PrintWriter(new File(fileName))) {
            StringBuilder sb = new StringBuilder();
            sb.append("Name,Price,Rating,Reviews,Availability,Image URL\n");
            for (Product product : products) {
                sb.append(product.getName()).append(",")
                  .append(product.getPrice()).append(",")
                  .append(product.getRating()).append(",")
                  .append(product.getReviews()).append(",")
                  .append(product.getAvailability()).append(",")
                  .append(product.getImageUrl()).append("\n");
            }
            writer.write(sb.toString());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // Clean up resources
    public void close() {
        if (driver != null) {
            driver.quit();
        }
    }
}

// Main class
public class Main {
    public static void main(String[] args) {
        AmazonScraper scraper = new AmazonScraper();
        List<Product> products = scraper.searchProducts("laptop", 2); // Example: Search for laptops, 2 pages
        scraper.saveToCSV(products, "products.csv");
        scraper.close();
    }
}
