---
title: 5. Profile picture for contact
weight: 750
ShowToc: true
TocOpen: true
---

So far in the second part, we have written the server side code to manage Contacts, no React code yet. We have developed almost all the server side code, but not dealt with images yet.

In this article you will learn how to upload profile picture for the contact.

## Where to Store Images?

We can store images in a web app at different locations. Some of the common storage options are

- Images folder inside the web app
- External server like Amazon S3 store, Cloudinary etc.
- Store in database

In our app, we chose to store all images in external service. Among the external service we chose [Cloudinary](https://cloudinary.com/).

If we store images inside a folder in the web app, the image data will take a lot of space. We also have to sync the images when we are executing the code on local, test or live environments. We might also need to optimize the image storage for caching and performance.

We did not choose database for storing images, because database like SQL Server are not meant to store and query image data.

We chose Cloudinary because it comes with a [free account](https://cloudinary.com/pricing). At the time of writing this article, it is 10th Feb, 2023, they allow 25 free monthly credits, which means 25k monthly image transformations and 25GB storage, which is good enough for a small sized startup or even small business. It is very helpful and suitable for developers, who can use Cloudinary for hosting images and test their apps. Amazon also has free tier, but Cloudinary is specialized in [image processing](https://cloudinary.com/documentation/image_transformations). Specially the auto detection of face from a picture is extremely useful feature for profile pictures.

![edit contact profile picture](/images/blog/edit-contact-profile-picture.jpg "edit contact profile picture")

## Image Fields in Contact Entity

In the Contact entity we have two fields for storing profile picture, PictureUrl and CloudinaryId. You can open Entities/Contact.cs and view the fields.

- PictureUrl contains the full url of the profile picture, it can be used in browser, web app or anywhere, it is the direct url of the image.
- CloudinaryId is the id generated from Cloudinary, it is helpful in deleting the image by id or doing transformations and other operations.

```cs
public class Contact
{
    [Key]
    public int ContactId { get; set; }
    [Required]
    public string? FirstName { get; set; }
    public string? PictureUrl { get; set; }
    public string? CloudinaryId { get; set; }
    // other fields
}
```

## Service for Image Upload

### Store Cloudinary Url secret in .env file

Go to cloudinary.com and create a new account and login. Go to the Cloudinary dashboard at https://console.cloudinary.com/console. There you should see your cloud name, api key and api secret. We need these 3 values in our service to work with the Cloudinary API.

Open your .env file, in the root of ASP.NET Web API project. It will have only one entry for SQLSERVER. Now update this file and add 3 more lines for the Cloudinary credentials as follows.

```cs
SQLSERVER=server=.\sqlexpress;database=AddressBook;Trusted_Connection=true;MultipleActiveResultSets=true;TrustServerCertificate=True;
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=xxxxx
CLOUDINARY_API_SECRET=xxxxx
```

### Update SecretUtility

We have a Utility class to load secret values from .env files. Open Common\SecretUtility.cs file and add three more properties here

```cs
public class SecretUtility
{
  public static string? SqlServer
  {
    get
    {
        return Environment.GetEnvironmentVariable("SQLSERVER");
    }
  }

  public static string? Cloudinary_Cloud_Name { 
    get
    {
        return Environment.GetEnvironmentVariable("CLOUDINARY_CLOUD_NAME");
    }
  }
  public static string? Cloudinary_API_Key
  {
    get
    {
        return Environment.GetEnvironmentVariable("CLOUDINARY_API_KEY");
    }
  }
  public static string? Cloudinary_API_Secret
  {
    get
    {
        return Environment.GetEnvironmentVariable("CLOUDINARY_API_SECRET");
    }
  }
}
```

### Interface for Cloudinary Service

We will create a service for uploading profile picture to Cloudinary. Create a new interface **ICloudinaryService** in Services folder and declare the following methods in it.

```cs
public interface ICloudinaryService
{
  CloudinaryUploadResultRes UploadProfilePictureThumbnail(IFormFile file, string tempFolderPath);
  void DeleteImage(string? cloudinaryId);
}
```

There are two methods.

- UploadProfilePictureThumbnail – It takes the IFormFile, which is a file sent with http post method. IFormFile is provided by ASP.NET Core. It also takes temp folder path, which will be provided by the Controller. It returns a Dto that has the URL of uploaded image and Cloudinary Id.
- DeleteImage – It just takes the Cloudinary Id

### Response Dto from Cloudinary Service

Lets create the Dto first. Create a new file **CloudinaryUploadResultRes** in **Dtos** folder and add the following code. It has two members.

- SecureUrl – The Url of the uploaded image on Cloudinary
- PublicId – Cloudinary Id

These two values are all we need to save in the database, in the Contact table.

```cs
public class CloudinaryUploadResultRes
{
  public string? SecureUrl { get; set; }
  public string? PublicId { get; set; }
}
```

### Cloudinary Service Implementation

Now we will go ahead and continue with the service implementation, which will use Cloudinary .NET SDK to upload profile picture of the contact. Create a new class CloudinaryService in Services folder and add the following code.

Make sure that CloudinaryDotNet official package is installed and updated with the latest version.

![nuget cloudinary](/images/blog/nuget-cloudinary.jpg "nuget cloudinary")

```cs
public class CloudinaryService : ICloudinaryService
{
    public CloudinaryService() {}

    public void DeleteImage(string? cloudinaryId)
    {
        Cloudinary cloudinary = new Cloudinary(new Account(
            CloudName, APIKey, APISecret));
        if (string.IsNullOrWhiteSpace(cloudinaryId) == false)
        {
            try
            {
                cloudinary.Destroy(new DeletionParams(cloudinaryId));
            }
            catch (Exception)
            {

            }
        }
    }

    private string? CloudName { get { return SecretUtility.Cloudinary_Cloud_Name; } }
    private string? APIKey { get { return SecretUtility.Cloudinary_API_Key; } }
    private string? APISecret { get { return SecretUtility.Cloudinary_API_Secret; } }

    public CloudinaryUploadResultRes UploadProfilePictureThumbnail(IFormFile file,
        string tempFolderPath)
    {
        var imagePath = saveNewFileInTempFolder(file, tempFolderPath);
        var uploadParams = new ImageUploadParams()
        {
            File = new FileDescription(imagePath),
            Folder = CloudinaryFolders.Profile,
            EagerTransforms = new List<Transformation>()
            {
                new EagerTransformation().Width(200).Height(200).Gravity("faces").Crop("thumb")
            }
        };
        Cloudinary cloudinary = new Cloudinary(new Account(
            CloudName, APIKey, APISecret));
        var result = cloudinary.Upload(uploadParams);

        File.Delete(imagePath);

        return new CloudinaryUploadResultRes
        {
            PublicId = result.PublicId,
            SecureUrl = result.Eager[0].SecureUrl.ToString()
        };
    }

    private string saveNewFileInTempFolder(IFormFile file, string pathToSave)
    {
        if (Directory.Exists(pathToSave) == false)
        {
            Directory.CreateDirectory(pathToSave);
        }

        if (file.Length > 0)
        {
            var fileName = Guid.NewGuid().ToString() + "." + Path.GetExtension(file.FileName);
            var fullPath = Path.Combine(pathToSave, fileName);
            using (var stream = new FileStream(fullPath, FileMode.Create))
            {
                file.CopyTo(stream);
            }
            return fullPath;
        }

        throw new Exception("Could not save profile picture");
    }
}
```

There are 3 private properties that load cloud name, api key and secret keys of Cloudinary from the .env file. These three values are required to initialize the Cloudinary class in the official SDK.

The UploadProfilePictureThumbnail method takes two parameters, an instance of ASP.NET Core’s IFromFile and temp folder path. We first save the image from IFormFile to a temp folder. Then we prepare the ImageUploadParams with the following.

- File – full path of image stored in temp folder
- Folder – the folder name in Cloudinary
- EagerTransforms – Update width and height to 200, auto detect face and crop the final image as thumbnail

This auto detection of face and method chaining in Cloudinary SDK is really helpful. Many image manipulations are done in a single line of code using method chaining.

After preparing the upload params, we initialize the SDK’s Cloudinary object with Account. Then we simply call the Upload method and pass the params and get the result. The temp image is deleted locally. We just need the SecureUrl and Cloudinary Id, we copy these from the result to the Dto and return the Dto.

In Cloudinary cloud storage, you can manage all your images in different folders. In the above code, we used CloudinaryFolders.Profile in cloud folder path. To define this path, create a new class CloudinaryFolders in Common.

```cs
public class CloudinaryFolders
{
  internal const string Profile = "addressbook/profile";
  internal const string General = "addressbook/general";
}
```

You can get the full code from [GitHub](https://github.com/saqibrazzaq/efcorebeginner/blob/main/AddressBook/AddressBook/Services/CloudinaryService.cs).

## Update Profile Picture Method in Contact Service

Now we can come back to the Contact Service and create method for updating the profile picture. Open Service\ContactService.cs and add the UpdateImage method as follows.

The method takes contactId, IFormFile and the temp folder path in parameters. It calls CloudinaryService.UploadProfilePictureThumbnail method and returns the response dto. We then find the contact from the repository, current record. We need existing record to get the existing Cloudinary Id of the image, we need it to delete the existing image from cloudinary. Once the existing image is deleted, we save the new Cloudinary Id and Url of the image in the Contact Entity. Finally we call the Save method to save the changes in the repository. This way we can update the profile picture of an existing Contact in our app, using Cloudinary as 3rd party cloud storage.

```cs
public class ContactService : IContactService
{
    private readonly IRepositoryManager _repositoryManager;
    private readonly IMapper _mapper;
    private readonly ICloudinaryService _cloudinaryService;
    // other code omitted

    public ContactRes UpdateImage(int contactId, IFormFile file, string tempFolderPath)
    {
        var uploadResult = _cloudinaryService.UploadProfilePictureThumbnail(file, tempFolderPath);

        var entity = FindContactIfExists(contactId, true);

        _cloudinaryService.DeleteImage(entity.CloudinaryId);

        entity.CloudinaryId = uploadResult.PublicId;
        entity.PictureUrl = uploadResult.SecureUrl;

        _repositoryManager.Save();
        return _mapper.Map<ContactRes>(entity);
    }
}
```