---
layout: post
title:  "JSON Web Tokens in django application - part I"
date:   2017-08-20
author:     "Kyle Ding"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags: 
    - Python
    - Django
---

During my work with the team, i learned a lot from them. Especially with CTO and my friends there. They taught me by giving a test code that is so beautiful. With Django Factory Boy I can create a shadow object during the testing phase without needing to create objects from the queryset that make it so simple.


------

This note gives me feedback in the future. I keep using this note for my reference in the code testing phase in Django Framework.

For installation, it’s pretty easy. I only need a package named `factory_boy`which includes the `Faker` package.

```shell
(myenv) $ pip install factory_boy
```

If you want to see full reference from factory_boy, you can visit this [link](http://factoryboy.readthedocs.io/en/latest/index.html).

### How to use ?

How to use it is quite easy. In this case I have a simple model class, called `Product` model.

```python
class Product(models.Model):
    sku = models.CharField(max_length=30, unique=True, 
                           primary_key=True)
    name = models.CharField(max_length=30)
    base_price = models.PositiveIntegerField()
    price = models.PositiveIntegerField()
    stock = models.PositiveIntegerField()
    stock_min = models.PositiveIntegerField()

    def __str__(self):
        return self.sku

    class Meta:
        db_table = 'product'
```

Then, iwill create a `tests` directory. However, I need to delete the `tests.py`file inside my app before making it.

Then I will create a structure like this inside my app:

```shell
myapp/
    tests/
        factories.py
        test_products_v1.py
```

The `factories.py` file contains the factory product that will create the product object as long as I call it in the testing phase. It will create as many objects as I want in testing. While the `test_products_v1.py` file contains the test code for the product.

The contents of the `factories.py` file are as follows:

```python
import factory
import random
from factory.django import DjangoModelFactory

class ProductFactory(DjangoModelFactory):
    @factory.sequence
    def sku(n):
        return 'SKU-%d' % n

    @factory.sequence
    def name(n):
        return 'PRODUCT-%d' % n

    base_price = factory.lazy_attribute(
                 lambda o: random.randint(100, 2000))
    price = factory.lazy_attribute(
            lambda o: random.randint(1000, 20000))
    stock = factory.lazy_attribute(lambda o: random.randint(1, 100))
    stock_min = factory.lazy_attribute(
                lambda o: random.randint(1, 5))

    class Meta:
        model = myapp.Product'
```

Look at the fields. It’s like a field on a model that’s in the `Product` model. In the `Meta` class, there is one attribute named `model`. This attribute contains a value referencing my Product model that has the format `<appname>. <ModelName>`.

Ok, if we make the test in the file `test_products_v1.py`, more or less we will use the Factory class we have created earlier:

```python
import json
from django.test import TestCase
from rest_framework.test import APITestCase
from rest_framework import status
from salesapi.tests.factories import ProductFactory
from sales.models import Product

class TestProductsV1(APITestCase):

    def test_product_list(self):
        for _ in range(3):
            ProductFactory()
        response = self.client.get(
            '/api-sales/products/'
        )
        expected = 3
        self.assertEqual(
            response.status_code, status.HTTP_200_OK)
        self.assertEqual(
            len(json.loads(response.content.decode('utf-8'))), 
            expected)
```

Simple and so elegant. I love it. Django & Python is fucking awesome.