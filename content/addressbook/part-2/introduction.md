---
title: 0. AddressBook - Part 2 – Contact Management
weight: 700
ShowToc: true
TocOpen: true
---

In the first part we built React frontend for managing city, state and countries. The backed API was built in ASP.NET Core web API. We created 5 entities, which have their own Repository, dtos, controller and services.

1. Country
2. State
3. City
4. Translation
5. Timezone

Now in the second part, we will develop the contact manager app. It is inspired from [Google Contacts](https://contacts.google.com/) web app. We will build the following features in our address book.

- Create, edit, delete contacts
- Search contacts using free text, it will search database in name, department, company, phone, email, website, address etc.
- Contact list will have paging support
- A contact may have multiple (0 or more) phone numbers, emails, chats, addresses, labels and websites
- Each phone number, email, chat, address etc. will have optional label, just like Google contacts app
- Upload profile picture for contact and display in contact list and edit contact page
- Left side menu will be dynamically loaded with all labels. Clicking on each label will load contacts with the specific label
- Delete contact will also delete contact’s child records e.g. contact’s phone, chat, website, addresses etc.
- Contact’s address contains CityId, which was created in first part. Deleting city will display number of addresses. You cannot delete a city, if it is used in an address.

![contacts main page](/images/blog/contacts-main-page.jpg "contacts main page")