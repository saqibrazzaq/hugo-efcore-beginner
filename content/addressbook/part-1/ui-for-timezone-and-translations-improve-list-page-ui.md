---
title: 11. UI for Timezone and Translations, Improve list page UI
weight: 620
ShowToc: true
TocOpen: true
---

In previous articles we created UI for Country, State and City. In each article we introduced and learned something new. This will be the last article of part one, in which we will create CRUD app having 5 entities.

## Timezone and Translation CRUD pages

The CRUD pages for Timezone and Translation will be developed in the same pattern of State pages. A Country may have multiple states. Similarly a Country may have multiple Timezones and multiple Translations. Country has one to many relationship with State, Timezone and Translation. We will just link the GitHub link of Timezone and Translation pages and will not go in detail.

- [Timezone folder having search, create/edit and delete pages](https://github.com/saqibrazzaq/efcorebeginner/tree/main/AddressBook/react-client/src/pages/timezone)
- [Translation folder](https://github.com/saqibrazzaq/efcorebeginner/tree/main/AddressBook/react-client/src/pages/translation)

And of course we will also update App.tsx to define the routing. The Url format will be exactly as the state page as below

- localhost:3000/timezones – list and search timezones
- localhost:3000/timezones/edit – create new timezone, country dropdown is not selected
- localhost:3000/timezones/edit/{countryId} – create new timezone, country dropdown is selected
- localhost:3000/timezones/edit/{countryId}/{timezoneId} – update an existing timezone, country is selected

## Menu item for Timezone and Translation pages

Now where should we add links to timezone and translation pages. In the top menu, we have links for Countries, States and Cities. We can directly open these pages by clicking on top menu link. Should we add Timezones and Translations in top menu? That would not be suitable, we cannot add everything in top menu. Top menu is just for the most important and commonly used pages.

There is one to many relationship between Country and Timezone/Translation. We can add in Create/Edit Country page. Or we can also add in Country list page. Both are suitable. We will add the links in Country list page as follows.

![country list with icons](/images/country-list-with-icons.jpg "country list with icons")

The above screenshot shows that we have 4 icon buttons, for each country. We had text links for Edit and Delete in all pages. If we add text links for Timezones and Translation, it will cover up a lot of space. We used icon buttons to save some space. We can also have a single icon button like option, more or three dots, which opens a popup menu. The popup menu can show edit, delete, timezone and translation for each country. We will add such menu, where there are more options for Country. For this article, we will continue with the four icon buttons, right on the country row.

## How to Add Icon Button

First lets review how we added text based links. We used Link component from Chakra UI. We render it as Link component from react-router-dom. Both have same name, so we rename React Router’s Link component to RouteLink. We just add text Edit, the link becomes text based.

```react
import {Link as RouteLink} from "react-router-dom";

<Link mr={2} as={RouteLink} to={"/countries/edit/" + item.countryId}>
  Edit
</Link>
```

We just replace text Edit with icon button to render the link as icon. We will create a new component for an icon button, and use this in every page. Lets create an icon button for Edit. Create new folder src\components\icons. Add a new file UpdateIcon.tsx. Type rafce and insert the default template code. Update the file as follows. We have used icon from [react icons](https://react-icons.github.io/react-icons/). You can easily search the icon of your choice and use it in Chakra UI’s IconButton component. We have also added tooltip.

```react
import { IconButton, Tooltip } from '@chakra-ui/react'
import { AiFillEdit } from "react-icons/ai";

const UpdateIcon = () => {
  return (
    <Tooltip label="Edit">
    <IconButton
      variant="outline"
      size="sm"
      fontSize="18px"
      colorScheme="blue"
      icon={<AiFillEdit />}
      aria-label="Edit"
    />
    </Tooltip>
  )
}

export default UpdateIcon
```

This Edit icon button is now a reusable component. We reuse it anywhere with minimal code. Lets edit Countries.tsx and replace Edit text with the Edit icon button. We have used this icon in all list pages e.g. countries, states, cities. It is very easier this way, to reuse from custom component. If we want to change something in the tooltip or choose a different icon from react-icons, we can do at one place, and it will be updated everywhere.

```react
<Link mr={2} as={RouteLink} to={"/countries/edit/" + item.countryId}>
  <UpdateIcon />
</Link>
```

Part one is now complete. You can get the complete code from this [GitHub repository](https://github.com/saqibrazzaq/efcorebeginner/tree/main/AddressBook).

The Address Book app is a small/medium level application, where we will have 8-10 entities, with relationships. We will use LINQ instead of SQL to read database using Entity Framework. In part one, we have 5 entities so far, we used Include methods to get relational data. In next part, we will have some more entities to manage addresses and some optimized and complex queries. In the last part, we will feed one million records to the database and see how our small application handles it. After a million records, I think it is worth to call it a medium scale application.