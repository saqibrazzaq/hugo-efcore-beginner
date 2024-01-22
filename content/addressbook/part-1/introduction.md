---
title: Part 1 – City, State and Country Management
weight: 510
ShowToc: true
TocOpen: true
---

In the first part, we will build API and UI for city, state and country management. These three are part of any address and should be used as foreign keys in the address table.

We will create a web API in ASP.NET Core 7 for backend. It will have repositories, dtos, services and controllers. Each entity will have its own repository, service, dtos and controller. In the [beginner ASP.NET series](/person-api), we had only one entity. For managing city, state and country, we just have more entities, but with few more features like

Create search dropdown boxes for city, state and country
Search form with dropdowns e.g. search all states by selecting a Country from the dropdown
In edit form, choose foreign key from dropdown, e.g. In state edit form, we need to assign it a country id. We can’t just take country id input in a textbox. We will add a Country dropdown in state edit form.
Prevent deleting parent entity when it has children. For example you should not be able to delete a Country, when it has states. Because those states will have cities as children. And city id might be used in other tables.