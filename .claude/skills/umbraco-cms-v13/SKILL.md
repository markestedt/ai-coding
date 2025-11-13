---
name: umbraco-cms-v13
description: Expert guidance for Umbraco CMS v13+ development including document types, compositions, properties, services, dependency injection, and best practices. Use when working with Umbraco CMS, content modeling, document types, compositions, or when user mentions Umbraco, CMS 13, or Umbraco content management.
---

# Umbraco CMS v13+ Development

Expert guidance for building and maintaining Umbraco CMS v13+ applications with .NET 7+.

## Core Concepts

### Document Types
Umbraco uses a flexible content model based on Document Types (similar to classes):

- **Document Types**: Define the structure of content pages
- **Compositions**: Reusable sets of properties (like interfaces/mixins)
- **Element Types**: Used for nested content and block list items
- **Media Types**: Define structure for media items

### Content Models
Umbraco generates strongly-typed models from Document Types:

```csharp
// Generated model (auto-generated in ~/umbraco/models/)
public partial class ArticlePage : PublishedContentModel
{
    // Property accessors
    public string Heading => this.Value<string>("heading");
    public IHtmlEncodedString BodyText => this.Value<IHtmlEncodedString>("bodyText");
    public DateTime PublishedDate => this.Value<DateTime>("publishedDate");
    public IPublishedContent FeaturedImage => this.Value<IPublishedContent>("featuredImage");
}
```

## Creating Document Types

### Using Compositions

```csharp
// Composition for SEO properties
public class SeoComposition
{
    public string MetaTitle { get; set; }
    public string MetaDescription { get; set; }
    public string MetaKeywords { get; set; }
}

// Composition for page settings
public class PageSettingsComposition
{
    public bool HideFromNavigation { get; set; }
    public string PageClass { get; set; }
}

// Document Type using compositions
public class ArticlePage : SeoComposition, PageSettingsComposition
{
    public string Heading { get; set; }
    public IHtmlEncodedString BodyText { get; set; }
    public DateTime PublishedDate { get; set; }
    public MediaWithCrops FeaturedImage { get; set; }
}
```

### Property Editors

```csharp
// Common property types

// Text string
[Display(Name = "Page Title")]
public string PageTitle { get; set; }

// Rich Text Editor
public IHtmlEncodedString BodyText { get; set; }

// Date/Time
public DateTime PublishedDate { get; set; }

// True/False
public bool IsPublished { get; set; }

// Numeric
public int ViewCount { get; set; }

// Content Picker (single)
public IPublishedContent RelatedPage { get; set; }

// Content Picker (multiple)
public IEnumerable<IPublishedContent> RelatedPages { get; set; }

// Media Picker
public MediaWithCrops FeaturedImage { get; set; }

// Multiple Media Picker
public IEnumerable<MediaWithCrops> Gallery { get; set; }

// Block List
public BlockListModel ContentBlocks { get; set; }

// Block Grid
public BlockGridModel PageLayout { get; set; }
```

## Block List & Block Grid

### Block List Item (Element Type)

```csharp
// Element Type for a Hero Block
public class HeroBlock
{
    public string Heading { get; set; }
    public string SubHeading { get; set; }
    public MediaWithCrops BackgroundImage { get; set; }
    public Link CallToAction { get; set; }
}

// Using in a view
@if (Model.ContentBlocks != null)
{
    foreach (var block in Model.ContentBlocks)
    {
        if (block.Content is HeroBlock heroBlock)
        {
            <div class="hero" style="background-image: url('@heroBlock.BackgroundImage.Url()')">
                <h1>@heroBlock.Heading</h1>
                <p>@heroBlock.SubHeading</p>
                @if (heroBlock.CallToAction != null)
                {
                    <a href="@heroBlock.CallToAction.Url" class="btn">
                        @heroBlock.CallToAction.Name
                    </a>
                }
            </div>
        }
    }
}
```

### Block Grid

```csharp
// Block Grid for advanced layouts
@if (Model.PageLayout != null)
{
    foreach (var row in Model.PageLayout)
    {
        <div class="grid-row" data-columns="@row.ColumnSpan">
            @foreach (var area in row.Areas)
            {
                <div class="grid-area" data-area="@area.Alias">
                    @foreach (var block in area)
                    {
                        @await Html.PartialAsync("BlockGrid/" + block.Content.ContentType.Alias, block)
                    }
                </div>
            }
        </div>
    }
}
```

## Services & Dependency Injection

### Built-in Services

```csharp
public class ArticleController : RenderController
{
    private readonly IPublishedContentQuery _publishedContentQuery;
    private readonly IContentService _contentService;
    private readonly IMediaService _mediaService;
    private readonly ILogger<ArticleController> _logger;
    private readonly IUmbracoContextAccessor _umbracoContextAccessor;

    public ArticleController(
        ILogger<ArticleController> logger,
        ICompositeViewEngine compositeViewEngine,
        IUmbracoContextAccessor umbracoContextAccessor,
        IPublishedContentQuery publishedContentQuery,
        IContentService contentService,
        IMediaService mediaService)
        : base(logger, compositeViewEngine, umbracoContextAccessor)
    {
        _logger = logger;
        _umbracoContextAccessor = umbracoContextAccessor;
        _publishedContentQuery = publishedContentQuery;
        _contentService = contentService;
        _mediaService = mediaService;
    }

    public override IActionResult Index()
    {
        var model = CurrentPage as ArticlePage;
        return View(model);
    }
}
```

### Custom Services

```csharp
// Define interface
public interface IArticleService
{
    IEnumerable<ArticlePage> GetRecentArticles(int count);
    IEnumerable<ArticlePage> GetRelatedArticles(ArticlePage article);
}

// Implement service
public class ArticleService : IArticleService
{
    private readonly IPublishedContentQuery _contentQuery;
    private readonly IUmbracoContextAccessor _umbracoContextAccessor;

    public ArticleService(
        IPublishedContentQuery contentQuery,
        IUmbracoContextAccessor umbracoContextAccessor)
    {
        _contentQuery = contentQuery;
        _umbracoContextAccessor = umbracoContextAccessor;
    }

    public IEnumerable<ArticlePage> GetRecentArticles(int count)
    {
        return _contentQuery
            .ContentAtRoot()
            .DescendantsOrSelfOfType<ArticlePage>()
            .Where(x => x.PublishedDate <= DateTime.Now)
            .OrderByDescending(x => x.PublishedDate)
            .Take(count);
    }

    public IEnumerable<ArticlePage> GetRelatedArticles(ArticlePage article)
    {
        // Custom logic for finding related articles
        return _contentQuery
            .ContentAtRoot()
            .DescendantsOrSelfOfType<ArticlePage>()
            .Where(x => x.Id != article.Id)
            .Take(3);
    }
}

// Register in Startup.cs or Composer
public class ServiceComposer : IComposer
{
    public void Compose(IUmbracoBuilder builder)
    {
        builder.Services.AddScoped<IArticleService, ArticleService>();
    }
}
```

## Querying Content

### Basic Queries

```csharp
// Get root content
var root = _contentQuery.ContentAtRoot();

// Get by ID
var content = _umbracoContext.Content.GetById(1234);

// Get by GUID
var content = _umbracoContext.Content.GetById(new Guid("12345678-1234-1234-1234-123456789012"));

// Get children
var children = content.Children;

// Get descendants
var descendants = content.Descendants();

// Type-specific queries
var articles = content.DescendantsOrSelfOfType<ArticlePage>();

// Filtering
var recentArticles = content
    .DescendantsOrSelfOfType<ArticlePage>()
    .Where(x => x.PublishedDate >= DateTime.Now.AddMonths(-1))
    .OrderByDescending(x => x.PublishedDate);
```

### Advanced Queries with IPublishedContentQuery

```csharp
// Search
var searchResults = _contentQuery.Search("search term");

// Content at XPath
var contentByXPath = _contentQuery.ContentAtXPath("//ArticlePage");

// Get single item
var singleItem = _contentQuery.ContentSingleAtXPath("//ArticlePage[@id='1234']");
```

## Routing & URLs

### Getting URLs

```csharp
// Get URL for content
var url = content.Url();

// Get absolute URL
var absoluteUrl = content.Url(mode: UrlMode.Absolute);

// Get URL for media
var mediaUrl = media.Url();

// Get cropped image URL
var croppedUrl = media.GetCropUrl("hero");
```

### Custom Routes

```csharp
// Route hijacking with custom controller
public class ArticlePageController : RenderController
{
    public ArticlePageController(
        ILogger<RenderController> logger,
        ICompositeViewEngine compositeViewEngine,
        IUmbracoContextAccessor umbracoContextAccessor)
        : base(logger, compositeViewEngine, umbracoContextAccessor)
    {
    }

    // Custom action for article detail
    public IActionResult ArticleDetail(ArticlePage model, string slug)
    {
        // Custom logic
        return View("ArticleDetail", model);
    }
}
```

### Virtual Routes

```csharp
public class CustomRouteComposer : IComposer
{
    public void Compose(IUmbracoBuilder builder)
    {
        builder.Services.Configure<UmbracoPipelineOptions>(options =>
        {
            options.AddFilter(new UmbracoPipelineFilter("CustomRoute")
            {
                Endpoints = app => app.UseEndpoints(endpoints =>
                {
                    endpoints.MapControllerRoute(
                        "custom-route",
                        "custom/{action}/{id?}",
                        new { controller = "Custom" });
                })
            });
        });
    }
}
```

## Notifications (Events)

### Content Notifications

```csharp
public class ContentNotificationHandler :
    INotificationHandler<ContentPublishingNotification>,
    INotificationHandler<ContentPublishedNotification>
{
    private readonly ILogger<ContentNotificationHandler> _logger;

    public ContentNotificationHandler(ILogger<ContentNotificationHandler> logger)
    {
        _logger = logger;
    }

    public void Handle(ContentPublishingNotification notification)
    {
        foreach (var item in notification.PublishedEntities)
        {
            // Validate before publishing
            if (item.ContentType.Alias == "articlePage")
            {
                var heading = item.GetValue<string>("heading");
                if (string.IsNullOrEmpty(heading))
                {
                    notification.CancelOperation(
                        new EventMessage("Validation", "Heading is required", EventMessageType.Error));
                }
            }
        }
    }

    public void Handle(ContentPublishedNotification notification)
    {
        foreach (var item in notification.PublishedEntities)
        {
            _logger.LogInformation($"Content published: {item.Name}");
            // Custom logic after publishing
        }
    }
}

// Register in Composer
public class NotificationComposer : IComposer
{
    public void Compose(IUmbracoBuilder builder)
    {
        builder.AddNotificationHandler<ContentPublishingNotification, ContentNotificationHandler>();
        builder.AddNotificationHandler<ContentPublishedNotification, ContentNotificationHandler>();
    }
}
```

## Data Access (Content/Media Service)

### Content Service (Management API)

```csharp
// Create content
var parentId = -1; // Root
var content = _contentService.Create("New Article", parentId, "articlePage");
content.SetValue("heading", "My Article");
content.SetValue("bodyText", "<p>Article content</p>");
_contentService.SaveAndPublish(content);

// Update content
var existing = _contentService.GetById(1234);
if (existing != null)
{
    existing.SetValue("heading", "Updated Heading");
    _contentService.SaveAndPublish(existing);
}

// Delete content
var result = _contentService.Delete(content);

// Move content
_contentService.Move(content, newParentId);

// Copy content
var copy = _contentService.Copy(content, newParentId, false);
```

### Media Service

```csharp
// Upload media
public int UploadMedia(Stream fileStream, string fileName, int parentId)
{
    var media = _mediaService.CreateMedia(fileName, parentId, "Image");

    media.SetValue(_mediaFileManager, _mediaUrlGeneratorCollection, _shortStringHelper,
        _contentTypeBaseServiceProvider, "umbracoFile", fileName, fileStream);

    _mediaService.Save(media);

    return media.Id;
}
```

## View Helpers & Extensions

### Common View Helpers

```razor
@using Umbraco.Cms.Web.Common.PublishedModels

@* Get property value *@
@Model.Heading

@* HTML encoded string *@
@Model.BodyText

@* Check if property has value *@
@if (Model.HasValue("featuredImage"))
{
    <img src="@Model.FeaturedImage.Url()" alt="@Model.FeaturedImage.Name" />
}

@* Get cropped image *@
<img src="@Model.FeaturedImage.GetCropUrl("hero")" alt="@Model.Heading" />

@* Recursive navigation *@
@await Html.PartialAsync("Navigation", Model.Root())
```

### Custom Extension Methods

```csharp
public static class PublishedContentExtensions
{
    public static string GetPageTitle(this IPublishedContent content)
    {
        return content.HasValue("pageTitle")
            ? content.Value<string>("pageTitle")
            : content.Name;
    }

    public static IEnumerable<IPublishedContent> GetVisibleChildren(this IPublishedContent content)
    {
        return content.Children
            .Where(x => !x.Value<bool>("hideFromNavigation"))
            .Where(x => x.IsVisible());
    }

    public static string GetMetaDescription(this IPublishedContent content)
    {
        if (content.HasValue("metaDescription"))
        {
            return content.Value<string>("metaDescription");
        }

        if (content.HasValue("bodyText"))
        {
            var bodyText = content.Value<IHtmlEncodedString>("bodyText").ToString();
            return bodyText.StripHtml().Truncate(160);
        }

        return string.Empty;
    }
}
```

## Best Practices

### 1. Use Compositions

```csharp
// Create reusable compositions for common property sets
// SEO, Navigation, Settings, etc.

// Instead of repeating properties on every document type,
// create compositions and compose document types from them

public class ArticlePage : SeoComposition, NavigationComposition
{
    // Article-specific properties only
    public string Heading { get; set; }
    public IHtmlEncodedString BodyText { get; set; }
}
```

### 2. Caching

```csharp
private readonly IAppPolicyCache _runtimeCache;

public ArticleService(IAppPolicyCache runtimeCache)
{
    _runtimeCache = runtimeCache;
}

public IEnumerable<ArticlePage> GetRecentArticles(int count)
{
    return _runtimeCache.GetCacheItem(
        $"recent-articles-{count}",
        () =>
        {
            // Expensive query
            return _contentQuery
                .ContentAtRoot()
                .DescendantsOrSelfOfType<ArticlePage>()
                .OrderByDescending(x => x.PublishedDate)
                .Take(count);
        },
        TimeSpan.FromMinutes(10));
}
```

### 3. Strongly-Typed Models

```csharp
// Enable ModelsBuilder in appsettings.json
{
  "Umbraco": {
    "CMS": {
      "ModelsBuilder": {
        "ModelsMode": "SourceCodeAuto",
        "ModelsDirectory": "~/umbraco/models"
      }
    }
  }
}

// Use strongly-typed models instead of dynamic
public IActionResult Index()
{
    var model = CurrentPage as ArticlePage; // Strongly-typed
    return View(model);
}

// Instead of
// var heading = CurrentPage.Value<string>("heading"); // Dynamic
```

### 4. Dependency Injection

```csharp
// Always use DI instead of static helpers
// Good
public class MyController : RenderController
{
    private readonly IArticleService _articleService;

    public MyController(IArticleService articleService)
    {
        _articleService = articleService;
    }
}

// Avoid
// var helper = new UmbracoHelper(); // Static, not testable
```

### 5. Content Security

```csharp
// Validate user permissions
var user = _backOfficeSecurityAccessor.BackOfficeSecurity.CurrentUser;
if (user != null)
{
    var content = _contentService.GetById(1234);
    var permissions = _contentService.GetPermissions(content);

    if (permissions.Any(x => x.UserId == user.Id && x.AssignedPermissions.Contains("U")))
    {
        // User can update
    }
}

// Sanitize user input
var sanitized = _htmlSanitizer.Sanitize(userInput);
```

## Multi-Language Support

### Working with Cultures

```csharp
// Get content in specific language
var culture = "en-US";
var content = _umbracoContext.Content.GetById(1234);
var heading = content.Value<string>("heading", culture: culture);

// Get all cultures
var cultures = content.Cultures;

// Check if content exists in culture
if (content.Cultures.ContainsKey("sv-SE"))
{
    var swedishHeading = content.Value<string>("heading", culture: "sv-SE");
}

// Create variant content
var content = _contentService.GetById(1234);
content.SetValue("heading", "Swedish Heading", "sv-SE");
_contentService.SaveAndPublish(content, "sv-SE");
```

### Language-aware Routing

```razor
@* Get URL in specific language *@
<link rel="alternate" hreflang="sv-SE" href="@Model.Url(culture: "sv-SE")" />

@* Current culture *@
@System.Globalization.CultureInfo.CurrentCulture.Name

@* Switch language *@
@foreach (var culture in Model.Cultures)
{
    <a href="@Model.Url(culture: culture.Key)">@culture.Value</a>
}
```

## Testing

### Unit Testing

```csharp
[TestClass]
public class ArticleServiceTests
{
    private Mock<IPublishedContentQuery> _contentQueryMock;
    private ArticleService _articleService;

    [TestInitialize]
    public void Setup()
    {
        _contentQueryMock = new Mock<IPublishedContentQuery>();
        _articleService = new ArticleService(_contentQueryMock.Object);
    }

    [TestMethod]
    public void GetRecentArticles_ReturnsCorrectCount()
    {
        // Arrange
        var mockArticles = CreateMockArticles(10);
        _contentQueryMock
            .Setup(x => x.ContentAtRoot())
            .Returns(mockArticles);

        // Act
        var result = _articleService.GetRecentArticles(5);

        // Assert
        Assert.AreEqual(5, result.Count());
    }
}
```

## Performance Optimization

### Output Caching

```csharp
// Enable output caching
services.AddOutputCache(options =>
{
    options.AddBasePolicy(builder => builder.Cache());
    options.AddPolicy("ArticleCache", builder =>
        builder.Cache()
            .Expire(TimeSpan.FromMinutes(10))
            .Tag("articles"));
});

// Use in controller
[OutputCache(PolicyName = "ArticleCache")]
public IActionResult Index()
{
    return View(CurrentPage);
}
```

### Examine (Search/Indexing)

```csharp
public class ArticleSearchService
{
    private readonly IExamineManager _examineManager;

    public ArticleSearchService(IExamineManager examineManager)
    {
        _examineManager = examineManager;
    }

    public IEnumerable<IPublishedContent> Search(string term)
    {
        if (_examineManager.TryGetIndex("ExternalIndex", out var index))
        {
            var searcher = index.Searcher;
            var results = searcher.CreateQuery("content")
                .NodeTypeAlias("articlePage")
                .And()
                .GroupedOr(new[] { "heading", "bodyText" }, term)
                .Execute();

            return results.Select(x => _umbracoContextAccessor
                .GetRequiredUmbracoContext()
                .Content
                .GetById(int.Parse(x.Id)));
        }

        return Enumerable.Empty<IPublishedContent>();
    }
}
```

## Version-Specific Features (v13+)

### .NET 7+ Support
- Uses .NET 7+ with minimal hosting model
- Top-level statements in Program.cs
- Enhanced performance

### Block Grid Editor
```csharp
// Advanced layout control with Block Grid
// Supports responsive layouts, areas, and nested structures
public BlockGridModel PageLayout { get; set; }
```

### Content Delivery API
```csharp
// Built-in headless API
// Configure in appsettings.json
{
  "Umbraco": {
    "CMS": {
      "DeliveryApi": {
        "Enabled": true,
        "PublicAccess": true
      }
    }
  }
}

// Access at: /umbraco/delivery/api/v1/content
```

### Improved ModelsBuilder
- Better performance
- Incremental compilation
- Enhanced type safety

## Troubleshooting

### Common Issues

1. **Models not generating**
   - Check ModelsBuilder mode in appsettings.json
   - Rebuild project
   - Check ~/umbraco/models directory permissions

2. **Content not appearing in frontend**
   - Verify content is published (not just saved)
   - Check template is assigned to document type
   - Verify view file exists in Views folder

3. **Route hijacking not working**
   - Controller name must match document type alias + "Controller"
   - Must inherit from RenderController
   - Check controller namespace is correct

4. **Property values returning null**
   - Verify property alias matches
   - Check property has a value in backoffice
   - Ensure property type is correct

## Configuration (appsettings.json)

```json
{
  "Umbraco": {
    "CMS": {
      "Global": {
        "Id": "your-umbraco-id",
        "SanitizeTinyMce": true
      },
      "Content": {
        "AllowEditInvariantFromNonDefault": false,
        "ContentVersionCleanupPolicy": {
          "EnableCleanup": true
        }
      },
      "ModelsBuilder": {
        "ModelsMode": "SourceCodeAuto",
        "ModelsDirectory": "~/umbraco/models",
        "AcceptUnsafeModelsDirectory": false
      },
      "DeliveryApi": {
        "Enabled": true,
        "PublicAccess": false
      }
    }
  }
}
```

## Resources

- Official Documentation: https://docs.umbraco.com/
- Community Forum: https://our.umbraco.com/
- Discord: https://discord.umbraco.com/
- .NET 7 Migration: Check Umbraco docs for v13 migration guide

## When to Use This Skill

Use this skill when:
- Creating or modifying Umbraco document types, compositions, or element types
- Working with Block List or Block Grid editors
- Implementing custom services and dependency injection
- Querying content with IPublishedContentQuery
- Setting up routing, notifications, or events
- Working with the Content/Media Service APIs
- Implementing caching, search, or performance optimizations
- Following Umbraco best practices and patterns
- Troubleshooting Umbraco-specific issues
