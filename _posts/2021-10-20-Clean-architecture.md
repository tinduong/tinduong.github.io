---
title: Skinny Controller
tags: [Clean architecture]
style: fill
color: secondary
description: Architect your solution for a clear/ clean implementation result in a super easy to navigate within the code base and unit test your implementation.
---

## Growing controller
When I started working with ASP.NET MVC back in 2014. Controller connect the Data that we need to pass into the UI for displaying the data. Most of the back end code and logic was store in the controller. However, it is getting bigger and ended up with 'FAT' controller with really complicated wired code... So There are approaches that help our controller code leaner and easier to manage all to the to an extreme to the point where controller is only as a gateway to the implementation and does not hold the responsibility of hosting code there. Migrating code to the different services will help you a lot with decouple everything and make it way easier to unit test

## Introducing services and mediatr
Creating services and mediatr will help you to migrate all the logic into more approritate places, more manageable and easier to maintain

I started to use mediatr and it is even better than just having a services and calling them from the controller. With mediatr, it will take care of the entire request. The concept is:

1. You create the event that can be 'handle'. This will carry the data that need to be used by the Handler
2. You create the EventHandler and this is where you will pass the data over to your services to be executed

Here is my skinny controller:
``` c#
 [HttpGet]
public async Task<ActionResult<ProductListViewModel>> GetAllProducts()
{
    var result = await _mediator.Send(new GetProductListRequest());
    return Ok(result);
}
```

As you can see, it is very clean. But what happens after this?
In my Core project I will setup the `GetProductListRequest` and `GetProductListRequestHandler`

```c#
public class GetProductlistQueryHandler : IRequestHandler<GetProductsListRequest, List<ProductListViewModel>>
{
    private readonly IAsyncRepository<Product> _productRepository;
    private readonly IMapper _mapper;

    public GetEventlistQueryHandler(IAsyncRepository<Product> productRepository, IMapper mapper)
    {
        _productRepository = productRepository;
        _mapper = mapper;
    }

    public async Task<List<ProductListViewModel>> Handle(GetProductsListRequest request, CancellationToken cancellationToken)
    {
        // handle with logic
        // do more with services that you can inject in here
        var allProduct = (await _productRepository.GetAllAsync()).OrderBy(item => item.ProductionDate);
        var result = _mapper.Map<List<ProductListViewModel>>(allProduct);
        return result;
    }
}

```

## Register your service using extension method:
In you core project (Not the API) set it up so that it can be register in your API

``` c#
public static class ApplicationServiceRegistration
{
    public static IServiceCollection AddApplicationService(this IServiceCollection services)
    {
        services.AddAutoMapper(Assembly.GetExecutingAssembly());
        services.AddMediatR(Assembly.GetExecutingAssembly());

        return services;
    }
}
```
When your API startup, don't forget that you need to register for it in your ConfigureServices

```
public void ConfigureServices(IServiceCollection services)
{
   ...

    // add services
    services.AddApplicationService();
    services.AddCommunicationService(Configuration);
    services.AddDatabaseServices(Configuration);

   ...
}
```
Take a quick look of your controller and now suddendly it is not a dumpster for code and you can unit test your RequestHandler and your services specifically.








