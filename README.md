import java.time.LocalDateTime;
import java.util.*;
import java.util.stream.Collectors;

// ------------------- Subscriber -------------------
abstract class Subscriber {
    private String id;
    private String name;
    private String email;
    private List<String> preferences;
    private String plan;

    public Subscriber(String id, String name, String email, List<String> preferences, String plan) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.preferences = preferences;
        this.plan = plan;
    }

    public String getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }

    public List<String> getPreferences() {
        return preferences;
    }

    public String getPlan() {
        return plan;
    }

    protected void setPlan(String plan) {
        this.plan = plan;
    }

    // Abstract method to build digest differently
    public abstract List<Article> buildDigest(List<Article> articles);

    @Override
    public String toString() {
        return name + " (" + plan + ")";
    }
}

// ------------------- PaidSubscriber -------------------
class PaidSubscriber extends Subscriber {
    private int maxArticles;

    public PaidSubscriber(String id, String name, String email, List<String> preferences) {
        super(id, name, email, preferences, "Paid");
        this.maxArticles = 10;
    }

    @Override
    public List<Article> buildDigest(List<Article> articles) {
        System.out.println("Building paid digest for " + getName());
        return articles.stream()
                .filter(a -> getPreferences().contains(a.getCategory()))
                .limit(maxArticles)
                .collect(Collectors.toList());
    }
}

// ------------------- FreeSubscriber -------------------
class FreeSubscriber extends Subscriber {
    private int maxArticles;

    public FreeSubscriber(String id, String name, String email, List<String> preferences) {
        super(id, name, email, preferences, "Free");
        this.maxArticles = 3;
    }

    @Override
    public List<Article> buildDigest(List<Article> articles) {
        System.out.println("Building free digest for " + getName());
        return articles.stream()
                .filter(a -> getPreferences().contains(a.getCategory()))
                .limit(maxArticles)
                .collect(Collectors.toList());
    }
}

// ------------------- Source -------------------
class Source {
    private String sourceId;
    private String name;
    private String category;
    private double trustScore;

    public Source(String sourceId, String name, String category, double trustScore) {
        this.sourceId = sourceId;
        this.name = name;
        this.category = category;
        this.trustScore = trustScore;
    }

    public String getSourceId() {
        return sourceId;
    }

    public String getName() {
        return name;
    }

    public String getCategory() {
        return category;
    }

    public double getTrustScore() {
        return trustScore;
    }

    @Override
    public String toString() {
        return name + " [" + category + "] Trust: " + trustScore;
    }
}

// ------------------- Article -------------------
class Article {
    private String articleId;
    private String title;
    private String category;
    private String publisher;
    private LocalDateTime publishTime;

    public Article(String articleId, String title, String category, String publisher, LocalDateTime publishTime) {
        this.articleId = articleId;
        this.title = title;
        this.category = category;
        this.publisher = publisher;
        this.publishTime = publishTime;
    }

    public String getArticleId() {
        return articleId;
    }

    public String getTitle() {
        return title;
    }

    public String getCategory() {
        return category;
    }

    public String getPublisher() {
        return publisher;
    }

    public LocalDateTime getPublishTime() {
        return publishTime;
    }

    @Override
    public String toString() {
        return title + " (" + category + ") from " + publisher + " at " + publishTime;
    }
}

// ------------------- NewsService -------------------
class NewsService {
    private List<Source> sources = new ArrayList<>();
    private List<Article> articles = new ArrayList<>();

    public void addSource(Source source) {
        sources.add(source);
    }

    public void fetchArticles() {
        System.out.println("Fetching articles...");
        articles.clear();
        for (Source src : sources) {
            // Simulating articles from sources
            for (int i = 1; i <= 2; i++) {
                articles.add(new Article(
                        src.getSourceId() + "-" + i,
                        src.getName() + " Article " + i,
                        src.getCategory(),
                        src.getName(),
                        LocalDateTime.now().minusHours(new Random().nextInt(48))
                ));
            }
        }
    }

    // Overloaded filter methods
    public List<Article> filter(String category) {
        return articles.stream()
                .filter(a -> a.getCategory().equalsIgnoreCase(category))
                .collect(Collectors.toList());
    }

    public List<Article> filter(LocalDateTime start, LocalDateTime end) {
        return articles.stream()
                .filter(a -> a.getPublishTime().isAfter(start) && a.getPublishTime().isBefore(end))
                .collect(Collectors.toList());
    }

    public List<Article> filter(String keyword, String category) {
        return articles.stream()
                .filter(a -> a.getTitle().toLowerCase().contains(keyword.toLowerCase()))
                .filter(a -> a.getCategory().equalsIgnoreCase(category))
                .collect(Collectors.toList());
    }

    public List<Article> getAllArticles() {
        return articles;
    }

    public void printSourceTrustReport() {
        System.out.println("\nSource Trust Report:");
        for (Source src : sources) {
            System.out.println(src);
        }
    }
}

// ------------------- NewsAppMain -------------------
public class NewsAppMain {
    public static void main(String[] args) {
        NewsService newsService = new NewsService();

        // Adding sources
        newsService.addSource(new Source("S001", "TechCrunch", "Tech", 9.5));
        newsService.addSource(new Source("S002", "ESPN", "Sports", 8.2));
        newsService.addSource(new Source("S003", "Engadget", "Tech", 7.8));
        newsService.addSource(new Source("S004", "Sky Sports", "Sports", 8.7));

        // Adding subscribers
        List<Subscriber> subscribers = new ArrayList<>();
        subscribers.add(new PaidSubscriber("U001", "Alice", "alice@example.com", Arrays.asList("Tech", "Sports")));
        subscribers.add(new FreeSubscriber("U002", "Bob", "bob@example.com", Arrays.asList("Tech")));
        subscribers.add(new FreeSubscriber("U003", "Charlie", "charlie@example.com", Arrays.asList("Sports")));

        // Simulate fetching articles
        newsService.fetchArticles();

        // Filter example: by category
        List<Article> techArticles = newsService.filter("Tech");
        List<Article> recentArticles = newsService.filter(LocalDateTime.now().minusDays(1), LocalDateTime.now());

        // Generate digests dynamically using polymorphism
        System.out.println("\nUser Digests:");
        for (Subscriber sub : subscribers) {
            List<Article> digest = sub.buildDigest(newsService.getAllArticles());
            System.out.println("\nDigest for " + sub.getName() + ":");
            digest.forEach(article -> System.out.println(" - " + article));
        }

        // Print source trust report
        newsService.printSourceTrustReport();
    }
}
