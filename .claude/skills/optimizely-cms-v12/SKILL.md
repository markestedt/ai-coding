---
name: optimizely-cms-v12
description: Expert guidance for Optimizely CMS v12+ development including content types, blocks, properties, routing, dependency injection, and best practices. Use when working with Optimizely CMS, content modeling, page types, blocks, or when user mentions Optimizely, CMS 12, or content management.
---

# Optimizely CMS v12+ Development

Expert guidance for building and maintaining Optimizely CMS v12+ applications with .NET 6+.

## Core Concepts

### Content Types
Optimizely uses strongly-typed content models that inherit from base classes:

- **PageData**: For page types
- **BlockData**: For block types (reusable components)
- **MediaData**: For media/file types

### Key Attributes

```csharp
[ContentType(
    DisplayName = "Article Page",
    GUID = "12345678-1234-1234-1234-123456789012",
    Description = "Used for news articles and blog posts")]
[ImageUrl("/icons/article.png")]
public class ArticlePage : PageData
{
    [Display(
        Name = "Main Heading",
        Description = "The main heading displayed at the top",
        GroupName = SystemTabNames.Content,
        Order = 10)]
    [Required]
    public virtual string Heading { get; set; }

    [Display(
        Name = "Main Body",
        GroupName = SystemTabNames.Content,
        Order = 20)]
    [UIHint(UIHint.Textarea)]
    public virtual XhtmlString MainBody { get; set; }

    [Display(
        Name = "Published Date",
        GroupName = SystemTabNames.Content,
        Order = 30)]
    public virtual DateTime? PublishedDate { get; set; }
}
```

## Common Property Types

### Built-in Property Types

```csharp
// Text
public virtual string Title { get; set; }

// Rich text
public virtual XhtmlString MainBody { get; set; }

// Number
public virtual int Count { get; set; }

// Date/Time
public virtual DateTime? PublishedDate { get; set; }

// Boolean
public virtual bool IsPublished { get; set; }

// URL
public virtual Url ExternalLink { get; set; }

// Content Reference (link to another page/block)
public virtual ContentReference RelatedPage { get; set; }

// Content Area (allows multiple blocks)
public virtual ContentArea MainContent { get; set; }

// Image Reference
[UIHint(UIHint.Image)]
public virtual ContentReference HeroImage { get; set; }

// Link Collection
public virtual LinkItemCollection Links { get; set; }
```

### Custom Property Types

```csharp
// Enumeration
[BackingType(typeof(PropertyNumber))]
public enum ArticleType
{
    News,
    BlogPost,
    PressRelease
}

public virtual ArticleType Type { get; set; }

// List of strings (using SelectMany)
[SelectMany(SelectionFactoryType = typeof(CategorySelectionFactory))]
public virtual string Categories { get; set; }
```

## Block Development

### Creating a Block

```csharp
[ContentType(
    DisplayName = "Hero Block",
    GUID = "87654321-4321-4321-4321-210987654321",
    Description = "Hero section with image and text")]
public class HeroBlock : BlockData
{
    [Display(Name = "Heading", Order = 10)]
    [Required]
    public virtual string Heading { get; set; }

    [Display(Name = "Sub Heading", Order = 20)]
    public virtual string SubHeading { get; set; }

    [Display(Name = "Background Image", Order = 30)]
    [UIHint(UIHint.Image)]
    public virtual ContentReference BackgroundImage { get; set; }

    [Display(Name = "Call To Action", Order = 40)]
    public virtual ButtonBlock CallToAction { get; set; }
}
```

### Block View (Razor)

```razor
@model HeroBlock

<div class="hero" style="background-image: url('@Url.ContentUrl(Model.BackgroundImage)')">
    <div class="hero-content">
        <h1>@Model.Heading</h1>
        @if (!string.IsNullOrEmpty(Model.SubHeading))
        {
            <p class="lead">@Model.SubHeading</p>
        }
        @Html.PropertyFor(m => m.CallToAction)
    </div>
</div>
```

## Dependency Injection & Services

### Registering Services

```csharp
// Startup.cs or Program.cs
services.AddSingleton<IArticleService, ArticleService>();
services.AddScoped<IContentRepository, ContentRepository>(); // Built-in
```

### Using Services in Controllers

```csharp
public class ArticlePageController : PageController<ArticlePage>
{
    private readonly IContentLoader _contentLoader;
    private readonly IUrlResolver _urlResolver;
    private readonly IArticleService _articleService;

    public ArticlePageController(
        IContentLoader contentLoader,
        IUrlResolver urlResolver,
        IArticleService articleService)
    {
        _contentLoader = contentLoader;
        _urlResolver = urlResolver;
        _articleService = articleService;
    }

    public IActionResult Index(ArticlePage currentPage)
    {
        var model = new ArticlePageViewModel(currentPage)
        {
            RelatedArticles = _articleService.GetRelatedArticles(currentPage)
        };

        return View(model);
    }
}
```

## Content Repository Pattern

### Loading Content

```csharp
// Get by ContentReference
var page = _contentLoader.Get<ArticlePage>(contentReference);

// Get children
var children = _contentLoader.GetChildren<ArticlePage>(parentReference);

// Get descendants
var descendants = _contentLoader.GetDescendents(startPageReference);

// Try get (safe)
if (_contentLoader.TryGet<ArticlePage>(contentReference, out var page))
{
    // Use page
}
```

### Querying Content

```csharp
// Using IContentRepository
var searchResults = _contentRepository
    .Search<ArticlePage>()
    .Filter(x => x.PublishedDate.After(DateTime.Now.AddMonths(-1)))
    .OrderBy(x => x.PublishedDate)
    .GetContentResult();

// Using Find (requires Optimizely Search & Navigation)
var results = _searchClient.Search<ArticlePage>()
    .Filter(x => x.Category.Match("Technology"))
    .Take(10)
    .GetContentResult();
```

## Routing & URLs

### Getting URLs

```csharp
// Get URL for content
var url = _urlResolver.GetUrl(contentReference);

// Get friendly URL
var friendlyUrl = _urlResolver.GetUrl(
    contentReference,
    language: "en",
    new VirtualPathArguments { ContextMode = ContextMode.Default });
```

### Custom Routing

```csharp
// Partial routing for custom URL segments
public class ArticlePartialRouter : IPartialRouter<ArticlePage, ArticlePageViewModel>
{
    public object RoutePartial(ArticlePage content, UrlResolverContext urlResolverContext)
    {
        var remainingSegments = urlResolverContext.RemainingSegments;

        if (remainingSegments.Count == 1)
        {
            // Custom logic for URL segment
            return new ArticlePageViewModel(content)
            {
                CustomParameter = remainingSegments[0]
            };
        }

        return null;
    }
}
```

## Best Practices

### 1. Content Type Design

- Use descriptive GUIDs (generate once, never change)
- Organize properties with Display attributes (Order, GroupName)
- Use appropriate UIHints for property rendering
- Add descriptions to help editors

### 2. Performance

```csharp
// Cache content when possible
private readonly IContentCacheKeyCreator _cacheKeyCreator;
private readonly ISynchronizedObjectInstanceCache _cache;

public ArticlePage GetCachedArticle(ContentReference reference)
{
    var cacheKey = _cacheKeyCreator.CreateCommonCacheKey(reference);

    return _cache.Get(cacheKey) as ArticlePage
        ?? _cache.Insert(cacheKey, _contentLoader.Get<ArticlePage>(reference));
}

// Use ContentReference instead of loading full content when possible
public virtual ContentReference RelatedPage { get; set; } // Good
// vs loading full page unnecessarily
```

### 3. Validation

```csharp
// Use data annotations
[Required(ErrorMessage = "Title is required")]
[StringLength(100, ErrorMessage = "Title cannot exceed 100 characters")]
public virtual string Title { get; set; }

// Custom validation
public class ArticlePageValidator : IValidate<ArticlePage>
{
    public IEnumerable<ValidationError> Validate(ArticlePage instance)
    {
        if (instance.PublishedDate > DateTime.Now)
        {
            yield return new ValidationError
            {
                PropertyName = nameof(instance.PublishedDate),
                ErrorMessage = "Published date cannot be in the future"
            };
        }
    }
}
```

### 4. Initialization Modules

```csharp
[InitializableModule]
[ModuleDependency(typeof(EPiServer.Web.InitializationModule))]
public class SiteInitialization : IInitializableModule
{
    public void Initialize(InitializationEngine context)
    {
        // Subscribe to events
        var contentEvents = context.Locate.ContentEvents();
        contentEvents.PublishingContent += OnPublishingContent;
    }

    private void OnPublishingContent(object sender, ContentEventArgs e)
    {
        if (e.Content is ArticlePage article)
        {
            // Custom logic before publishing
        }
    }

    public void Uninitialize(InitializationEngine context)
    {
        var contentEvents = context.Locate.ContentEvents();
        contentEvents.PublishingContent -= OnPublishingContent;
    }
}
```

## Security

### Content Access Rights

```csharp
// Check access
if (_contentSecurityRepository.HasAccess(contentReference, AccessLevel.Read))
{
    var content = _contentLoader.Get<IContent>(contentReference);
}

// Set access rights
var acl = _contentSecurityRepository.Get(contentReference).CreateWritableClone();
acl.AddEntry(new AccessControlEntry(
    "Everyone",
    AccessLevel.Read));
_contentSecurityRepository.Save(contentReference, acl, SecuritySaveType.Replace);
```

### Input Validation

```csharp
// Always validate user input
[ValidateInput(false)] // Only when needed for rich text
public IActionResult SubmitForm(FormModel model)
{
    if (!ModelState.IsValid)
    {
        return View(model);
    }

    // Sanitize HTML if accepting user input
    var sanitized = _htmlSanitizer.Sanitize(model.UserInput);

    // Process form
}
```

## Working with Media

### Upload and Store Media

```csharp
public ContentReference UploadImage(Stream imageStream, string fileName)
{
    var mediaFolder = _contentRepository.GetDefault<ImageFolder>(
        ContentReference.GlobalBlockFolder);

    var imageFile = _contentRepository.GetDefault<ImageFile>(mediaFolder.ContentLink);
    imageFile.Name = fileName;

    var blob = _blobFactory.CreateBlob(imageFile.BinaryDataContainer, ".jpg");
    blob.WriteAllBytes(imageStream.ToArray());
    imageFile.BinaryData = blob;

    return _contentRepository.Save(imageFile, SaveAction.Publish);
}
```

## Multi-language Support

### Language-aware Content

```csharp
// Get content in specific language
var page = _contentLoader.Get<ArticlePage>(
    contentReference,
    new LanguageSelector("sv"));

// Get all language versions
var languages = _contentLoader.Get<ArticlePage>(contentReference)
    .ExistingLanguages;

// Create language version
var writableClone = page.CreateWritableClone() as ArticlePage;
writableClone.Language = new CultureInfo("sv");
_contentRepository.Save(writableClone, SaveAction.Publish);
```

## Testing

### Unit Testing Content Types

```csharp
[TestClass]
public class ArticlePageTests
{
    private Mock<IContentLoader> _contentLoaderMock;
    private ArticlePageController _controller;

    [TestInitialize]
    public void Setup()
    {
        _contentLoaderMock = new Mock<IContentLoader>();
        _controller = new ArticlePageController(_contentLoaderMock.Object);
    }

    [TestMethod]
    public void Index_WithValidPage_ReturnsView()
    {
        // Arrange
        var page = new ArticlePage { Heading = "Test" };

        // Act
        var result = _controller.Index(page) as ViewResult;

        // Assert
        Assert.IsNotNull(result);
        Assert.IsInstanceOfType(result.Model, typeof(ArticlePageViewModel));
    }
}
```

## Common Patterns

### View Model Pattern

```csharp
public class ArticlePageViewModel : PageViewModel<ArticlePage>
{
    public ArticlePageViewModel(ArticlePage currentPage) : base(currentPage)
    {
    }

    public IEnumerable<ArticlePage> RelatedArticles { get; set; }
    public string AuthorName { get; set; }
    public int ReadingTime { get; set; }
}
```

### Repository Pattern for Business Logic

```csharp
public interface IArticleRepository
{
    IEnumerable<ArticlePage> GetRecentArticles(int count);
    IEnumerable<ArticlePage> GetArticlesByCategory(string category);
    ArticlePage GetArticleBySlug(string slug);
}

public class ArticleRepository : IArticleRepository
{
    private readonly IContentLoader _contentLoader;
    private readonly IContentRepository _contentRepository;

    public ArticleRepository(
        IContentLoader contentLoader,
        IContentRepository contentRepository)
    {
        _contentLoader = contentLoader;
        _contentRepository = contentRepository;
    }

    public IEnumerable<ArticlePage> GetRecentArticles(int count)
    {
        var startPage = _contentLoader.Get<StartPage>(ContentReference.StartPage);

        return _contentLoader.GetDescendents(startPage.ContentLink)
            .Select(x => _contentLoader.Get<IContent>(x))
            .OfType<ArticlePage>()
            .Where(x => x.PublishedDate.HasValue)
            .OrderByDescending(x => x.PublishedDate)
            .Take(count);
    }
}
```

## Troubleshooting

### Common Issues

1. **Content not appearing in edit mode**
   - Check ContentType GUID is unique
   - Verify inheritance from correct base class
   - Rebuild the project

2. **Properties not saving**
   - Ensure properties are `virtual`
   - Check validation rules
   - Verify property type is supported

3. **Routing issues**
   - Check IPartialRouter implementation
   - Verify route table configuration
   - Clear URL rewrite cache

4. **Performance problems**
   - Implement caching for frequently accessed content
   - Use ContentReference instead of loading full content
   - Optimize queries with proper filtering

## Version-Specific Features (v12+)

### .NET 6+ Integration
- Uses .NET 6 minimal hosting model
- Supports top-level statements in Program.cs
- Enhanced dependency injection

### Content Delivery API
```csharp
// Built-in REST API for headless scenarios
// Configure in Startup.cs
services.AddContentDeliveryApi();

// Access at: /api/episerver/v3.0/content
```

### Enhanced Search
```csharp
// Improved search with better performance
var results = _searchClient
    .Search<ArticlePage>()
    .For("search term")
    .InField(x => x.Heading)
    .InField(x => x.MainBody)
    .Take(20)
    .GetContentResult();
```

## Resources

- Official Documentation: https://docs.developers.optimizely.com/
- .NET 6 Migration Guide: Check Optimizely docs for CMS 12 migration
- Community Forum: https://world.optimizely.com/forum/

## When to Use This Skill

Use this skill when:
- Creating or modifying Optimizely content types (pages, blocks, media)
- Implementing custom routing or URL handling
- Working with the Content Repository API
- Setting up dependency injection for Optimizely services
- Implementing validation, caching, or security features
- Troubleshooting Optimizely-specific issues
- Following Optimizely best practices and patterns
