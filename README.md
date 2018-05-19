# Tapioca.HATEOAS
This is a simple library to implements HATEOAS docummentation in your RESTFul API's

-- Import with command line
```bash
Install-Package Tapioca.HATEOAS -Version 1.0.0
```

2 - Implements ISupportsHyperMedia in your exposed object.
```csharp
namespace RESTFulSampleServer.Data.VO
{
    public class BookVO : ISupportsHyperMedia
    {
        public long? Id { get; set; }
        public string Title { get; set; }
        public string Author { get; set; }
        public decimal Price { get; set; }
        public DateTime LaunchDate { get; set; }

        public List<HyperMediaLink> Links { get; set; } = new List<HyperMediaLink>();
    }
}
```

3 - Implements ISupportsHyperMedia in your exposed object.
```csharp
namespace RESTFulSampleServer.HyperMedia
{
    public class BookEnricher : ObjectContentResponseEnricher<BookVO>
    {
        protected override Task EnrichModel(BookVO content, IUrlHelper urlHelper)
        {
            var path = "api/books/v1";
            var url = new { controller = path, id = content.Id };

            content.Links.Add(new HyperMediaLink()
            {
                Action = HttpActionVerb.GET,
                Href = urlHelper.Link("DefaultApi", url),
                Rel = RelationType.self,
                Type = RensponseTypeFormat.DefaultGet
            });
            content.Links.Add(new HyperMediaLink()
            {
                Action = HttpActionVerb.POST,
                Href = urlHelper.Link("DefaultApi", url),
                Rel = RelationType.self,
                Type = RensponseTypeFormat.DefaultPost
            });
            content.Links.Add(new HyperMediaLink()
            {
                Action = HttpActionVerb.PUT,
                Href = urlHelper.Link("DefaultApi", url),
                Rel = RelationType.self,
                Type = RensponseTypeFormat.DefaultPost
            });
            content.Links.Add(new HyperMediaLink()
            {
                Action = HttpActionVerb.DELETE,
                Href = urlHelper.Link("DefaultApi", url),
                Rel = RelationType.self,
                Type = "int"
            });
            return null;
        }
    }
}
```
4 - Implements ISupportsHyperMedia in your exposed object.
```csharp
namespace RESTFulSampleServer.Controllers
{
    [ApiVersion("1")]
    [Route("api/[controller]/v{version:apiVersion}")]
    public class BooksController : Controller
    {
        private IBookBusiness _bookBusiness;

        public BooksController(IBookBusiness bookBusiness)
        {
            _bookBusiness = bookBusiness;
        }

        [HttpGet]
        //Add HyperMediaFilter
        [TypeFilter(typeof(HyperMediaFilter))]
        public IActionResult Get()
        {
            return new OkObjectResult(_bookBusiness.FindAll());
        }

        [HttpGet("{id}")]
        //Add HyperMediaFilter
        [TypeFilter(typeof(HyperMediaFilter))]
        public IActionResult Get(long id)
        {
            var book = _bookBusiness.FindById(id);
            if (book == null) return NotFound();
            return new OkObjectResult(book);
        }

        [HttpPost]
        //Add HyperMediaFilter
        [TypeFilter(typeof(HyperMediaFilter))]
        public IActionResult Post([FromBody]BookVO book)
        {
            if (book == null) return BadRequest();
            return new OkObjectResult(_bookBusiness.Create(book));
        }

        [HttpPut]
        //Add HyperMediaFilter
        [TypeFilter(typeof(HyperMediaFilter))]
        public IActionResult Put([FromBody]BookVO book)
        {
            if (book == null) return BadRequest();
            var updatedBook = _bookBusiness.Update(book);
            if (updatedBook == null) return BadRequest();
            return new OkObjectResult(updatedBook);
        }

        [HttpDelete("{id}")]
        //Add HyperMediaFilter
        [TypeFilter(typeof(HyperMediaFilter))]
        public IActionResult Delete(int id)
        {
            _bookBusiness.Delete(id);
            return NoContent();
        }
    }
}
```
4 - Implements ISupportsHyperMedia in your exposed object.
```csharp
    var filtertOptions = new HyperMediaFilterOptions();
    filtertOptions.ObjectContentResponseEnricherList.Add(new BookEnricher());
    filtertOptions.ObjectContentResponseEnricherList.Add(new PersonEnricher());
    services.AddSingleton(filtertOptions);
```