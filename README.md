# Découpage : Interfaces | Repositories | Services

## Pourquoi ce découpage ? 🤔

### Objectifs et Avantages

1. **Séparation des préoccupations (SoC)** :
    - **Interfaces** : Définissent ce que les classes doivent faire sans imposer comment elles le font.
    - **Repositories** : Gèrent l'accès aux données et les opérations CRUD.
    - **Services** : Contiennent la logique métier de l'application.

2. **Facilité de test** 🧪 :
    - Les interfaces et repositories permettent de créer des versions factices (mocks) pour les tests unitaires.
    - Tests rapides et fiables, indépendants de la base de données.

3. **Flexibilité et extensibilité** 🔧 :
    - Changer l'implémentation de l'accès aux données sans affecter les autres parties de l'application.
    - Ajouter de nouvelles implémentations de repositories si nécessaire.

4. **Maintenance** 🔄 :
    - Responsabilités clairement séparées.
    - Bugs plus faciles à localiser et corriger.

5. **Respect des principes SOLID** 📐 :
    - **SRP** : Une seule responsabilité par classe.
    - **OCP** : Classes ouvertes à l'extension mais fermées à la modification.
    - **LSP** : Les implémentations peuvent remplacer les classes de base sans affecter l'application.
    - **ISP** : Interfaces spécifiques aux besoins des clients.
    - **DIP** : Dépendance aux abstractions plutôt qu'aux implémentations concrètes.

## Exemple Sans découpage

```c#
public class ProductService
{
    private readonly MyDbContext _context;

    public ProductService(MyDbContext context)
    {
        _context = context;
    }

    public IEnumerable<Product> GetProductsInStockByCategory(string category)
    {
        return _context.Products
                   .Where(p => p.Category == category)
                   .Where(p => p.InStock > 0)
                   .ToList();
    }

}
```

```c#
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Category { get; set; }
    public int InStock { get; set; }
}
```

Problèmes :

- Couplage fort avec MyDbContext (dépendance directe à la base de données).
- Difficile à tester (pas de moyen simple de simuler MyDbContext).
- Pas de séparation claire des responsabilités.

## Exemple de découpage
### Interface du repository

1. Interface du repository
```c#
public interface IProductRepository
{
    IEnumerable<Product> GetProductsByCategory(string category);
}
```

2. Implémentation du repository
```c#
public class ProductRepository : IProductRepository
{
    private readonly MyDbContext _context;

    public ProductRepository(MyDbContext context)
    {
        _context = context;
    }

    public IEnumerable<Product> GetProductsByCategory(string category)
    {
        return _context.Products.Where(p => p.Category == category).ToList();
    }
}
```
3. Service
```c#
public class ProductService
{
    private readonly IProductRepository _productRepository;

    public ProductService(IProductRepository productRepository)
    {
        _productRepository = productRepository;
    }

    public IEnumerable<Product> GetProductsInStockByCategory(string category)
    {
        var products = _productRepository.GetProductsByCategory(category);
        return products.Where(p => p.InStock > 0);
    }
}
```

4. Test avec un repository factice

```c#
public class FakeProductRepository : IProductRepository
{
    private readonly List<Product> _products;

    public FakeProductRepository()
    {
        _products = new List<Product>
        {
            new Product { Id = 1, Name = "Product1", Category = "Category1", InStock = 10 },
            new Product { Id = 2, Name = "Product2", Category = "Category2", InStock = 0 },
            new Product { Id = 3, Name = "Product3", Category = "Category1", InStock = 5 }
        };
    }

    public IEnumerable<Product> GetProductsByCategory(string category)
    {
        return _products.Where(p => p.Category == category);
    }
}


public class ProductServiceTests
{
    [Fact]
    public void GetProductsInStockByCategory_ReturnsCorrectProducts()
    {
        // Arrange
        var fakeRepository = new FakeProductRepository();
        var productService = new ProductService(fakeRepository);

        // Act
        var result = productService.GetProductsInStockByCategory("Category1");

        // Assert
        Assert.Equal(2, result.Count()); // Il doit y avoir 2 produits en stock dans la catégorie "Category1"
        Assert.Contains(result, p => p.Name == "Product1");
        Assert.Contains(result, p => p.Name == "Product3");
    }
}

```

### Conclusion
Ce découpage permet de créer un code plus propre, maintenable et testable. Il facilite la gestion des dépendances, améliore la modularité de l'application, et respecte les principes de bonne conception logicielle.
