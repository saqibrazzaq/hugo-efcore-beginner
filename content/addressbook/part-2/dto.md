---
title: 2. DTOs for Contact and Related entities
weight: 720
ShowToc: true
TocOpen: true
---

After creating the entities and migrating the changes to SQL Server database, the next task is to create DTOs for the entities. In part one, we had 5 entities and in this part, we created 13 new entities. Lets start creating the DTOs now.

## DTOs for Label Definition Entities

We create 3 DTOs for each entity. One for response, one for create/edit and the last one for search request.

Create a new file PhoneLabel.cs in Dtos folder and add the following code. PhoneLabelRes is similar to PhoneLabel entity, but without EF Core related annotations. PhoneLabelReqEdit is for create/update, it does not have Id field. The Label field has Required and MaxLength validation attributes for built in ASP.NET data validation. And the PhoneLabelReqSearch inherits from PagedReq, which contains SearchText and paging related members.

```cs
public class PhoneLabelRes
{
  public int PhoneLabelId { get; set; }
  public string? Label { get; set; }
}

public class PhoneLabelReqEdit
{
  [Required, MaxLength(20)]
  public string? Label { get; set; }
}

public class PhoneLabelReqSearch : PagedReq
{

}
```

We have created such Dtos for Country, State, City. The Dtos for label definition entities are very simple, nothing special to write about. You may create the Dtos for other 5 Label definition entities on your own. Below is the list of all Label Dtos with GitHub links.

- AddressLabel
- ChatLabel
- EmailLabel
- Label
- PhoneLabel
- WebsiteLabel

## DTO for Contact

Contact is the main entity, that contains contact information like name, address, phone, email etc. Data that has one to one relationship with the contact are in the Contact table. Other data which has 0 to many relationship with the contact are in separate table, having ContactId as foreign key in these tables.

Create Contact.cs file in Dtos folder and add the following code.

```cs
public class ContactRes
{
  public int ContactId { get; set; }
  public string? FirstName { get; set; }
  public string? MiddleName { get; set; }
  public string? LastName { get; set; }
  public string? PictureUrl { get; set; }
  public string? Company { get; set; }
  public string? JobTitle { get; set; }
  public string? Department { get; set; }
  public DateTime DateOfBirth { get; set; }
  public string? Notes { get; set; }

  // Child tables
  public IEnumerable<ContactLabelRes>? ContactLabels { get; set; }
  public IEnumerable<ContactEmailRes>? ContactEmails { get; set; }
  public IEnumerable<ContactPhoneRes>? ContactPhones { get; set; }
  public IEnumerable<ContactAddressRes>? ContactAddresses { get; set; }
  public IEnumerable<ContactWebsiteRes>? ContactWebsites { get; set; }
  public IEnumerable<ContactChatRes>? ContactChats { get; set; }
}

public class ContactReqEdit
{
  [Required]
  public string? FirstName { get; set; }
  public string? MiddleName { get; set; }
  public string? LastName { get; set; }
  public string? PictureUrl { get; set; }
  public string? Company { get; set; }
  public string? JobTitle { get; set; }
  public string? Department { get; set; }
  public DateTime DateOfBirth { get; set; }
  public string? Notes { get; set; }
}

public class ContactReqSearch : PagedReq
{
  public int? LabelId { get; set; }
}
```

The ContactRes Dto is similar to the Contact entity. It also has list of child tables like addresses, emails, phones etc. You will get error on these child tables now, because these child tables are not created yet. We will create these soon.

The ContactReqEdit Dto is for create/edit contact. Phone, email, address etc cannot be created/updated using Contact Dto, they will be managed using their own Dtos.

ContactReqSearch inherits from PagedReq as all search Dtos. It has LabelId as its own member, so that we can search contacts by labels.

## DTOs for Phone, Email and Other Optional Data

There are 6 type of properties that are optional for a Contact. These are phone, email, address, label, chat and website. One contact may have 0 or multiple phone, email, address etc. Lets begin with ContactPhone dtos. Create a new file ContactPhone.cs in Dtos folder and write the code below.

```cs
public class ContactPhoneRes
{
  public int ContactPhoneId { get; set; }
  public string? Phone { get; set; }

  public int? CountryId { get; set; }
  public Country? Country { get; set; }

  public int ContactId { get; set; }
  public ContactRes? Contact { get; set; }

  public int? PhoneLabelId { get; set; }
  public PhoneLabelRes? PhoneLabel { get; set; }
}

public class ContactPhoneReqEdit
{
  [Required]
  public string? Phone { get; set; }
  // Foreign keys
  public int? CountryId { get; set; }
  [Required]
  public int? ContactId { get; set; }
  public int? PhoneLabelId { get; set; }
}

public class ContactPhoneReqSearch : PagedReq
{
  [Required]
  public int? ContactId { get; set; }
}
```

In ContactPhoneReqEdit dto, phone is required, one must enter a phone number to save it with the contact. Another required field is ContactId, without ContactId, the phone will belong to no one, so it is a must. Country and Label are optional, so have nullable integer types.

Lets create the Dtos for address as well, it has more fields and might seem complex. Create a new file ContactAddress.cs in Dtos folder and write the following code. ContactAddressRes is similar to the ContactAddress entity. ContactAddressEdit has two required fields.

- Line1
- ContactId

All the other fields are optional and can be left null or empty. One can just enter a street address in line1 field and save the address.

```cs
public class ContactAddressRes
{
  public int ContactAddressId { get; set; }
  public string? Line1 { get; set; }
  public string? Line2 { get; set; }
  public string? PostCode { get; set; }

  // Foreign keys
  public int? CityId { get; set; }
  public CityRes? City { get; set; }

  public int? AddressLabelId { get; set; }
  public AddressLabelRes? AddressLabel { get; set; }

  public int? ContactId { get; set; }
  public ContactRes? Contact { get; set; }
}

public class ContactAddressReqEdit
{
  [Required]
  public string? Line1 { get; set; }
  public string? Line2 { get; set; }
  public string? PostCode { get; set; }
  public int? CityId { get; set; }
  public int? AddressLabelId { get; set; }
  public int? ContactId { get; set; }
}

public class ContactAddressReqSearch : PagedReq
{
  [Required]
  public int? ContactId { get; set; }
}
```

You can write Dtos for the remaining four entities yourself. For reference you can check out the [GitHub repository](https://github.com/saqibrazzaq/efcorebeginner/tree/main/AddressBook/AddressBook/Dtos).