---
title: Segragation of Repository
tags: [Clean architecture, DB, Repository]
style: fill
color: primary
description: Stuff everything to your repository in the long term will make it looks like a chaos, Seperate your repository implementation!
---

## All in one
IProductRepository with 20 methods will quickly become a monster when you started to implement it in your ProductRepository. Instead we will break them down to at least 2 level

## The Master Interface


```C# 
public interface IAsyncRepository<T>: Where T: Class 
{
    Task<T> GetByIdAsync(Guid id);
    Task<IReadOnlyList<T>> GetAllAsync();
    Task<T> AddItemAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(T entity);
} 
```

## The Specific Interface

```c#
public interface IProductRepository: IAsyncRepository<Product> 
{
    Task<List<ProductByCategoryViewModel>> GetProductsByCategory(guid categoryId);
}

```

## The Base implementation

``` C#
public class BaseRepository<T>: IAsyncRepository<T> Where T: Class 
{
    // implementing CRUD
}

```

## The specific implementation
``` C#

public class ProductRepository: BaseRepository<Product>, IProductRepository 
{
    public async Task<List<ProductByCategoryViewModel>> GetProductsByCategory(guid categoryId) 
    {
        // implementation for this specific thing here
    }
}

```