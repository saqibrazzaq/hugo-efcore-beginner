---
title: Change DateTime Format in ASP.NET Core Web API for JSON
ShowToc: true
TocOpen: true
cover:
  image: "/images/blog/date-time-format.jpg"
  caption: "Photo by Panos Sakalakis on Unsplash"
---

## How .NET Core formats the date time?

Declare a variable of type DateTime in ASP.NET Core and print its value. What will you get?

```cs
DateTime dt = DateTime.Now;
Console.WriteLine(dt);
```

I got the following output in my local system. The format is dd/MM/yyyy hh:mm:ss tt

23/02/2023 10:50:03 am

![date time output](/images/blog/datetime-output-1024x190.jpg "date time output")

If you run the same program in your system, you might get a different output. Since we used DateTime.Now, we did not specify any locale or format, so it picks the default System locale and date time format.

I am on Windows 11 and the default DateTime.Now prints whatever I have set in my Windows 11 settings.

Lets try to do some experiment and change it to another format.

Open Settings in Windows 10/11, type date, it will show you options, choose “Set the current date and time formats”. It will show you Regional format, which will display the calendar type, date and time format. You will also see a dropdown on the right. This regional setting is applied in your System, which is used by all programs. .NET Core also uses this system settings.

Lets change is to English (United States) and run the program again. This settings has mm/dd/yyyy format. The AM/PM is in capital. hmmm. We had dd/mm/yyyy before and our am/pm was in small letters.

![date time english united states](/images/blog/date-time-english-united-states.jpg "date time english united states")

When I change to above settings in Windows, run my program, it shows the updated format now.

![english united states program output](/images/blog/english-united-states-program-output-1024x178.jpg "english united states program output")

Lets change it to English (Sweden). In Windows settings you will now see that it has yyyy-mm-dd format. There is no am/pm in time.

![english sweden](/images/blog/english-sweden.jpg "english sweden")

Lets run our C# program again, with the same default DateTime.Now output. As expected, it honors the Windows settings.

![english sweden program output](/images/blog/english-sweden-program-output-1024x176.jpg "english sweden program output")

## Different Output of Same Program on Local, Staging, Live systems

On local system, you may have English (United States) regional settings, the date time output on the local system will be mm/dd/yyyy.

When you deploy your program to live server, whose Operating System might be Linux, hosted in Europe, the regional settings might be different, it will output the date time in a different format.

The same program, might give different results, when executed on different systems.

The date/time value itself will be different. The date time format will be different.

**So what is the solution?**

For value, use DateTime.UtcNow. It will use GMT with 0 offset. Whenever someone reads, he should know that the date time value is UTC.

For format, if the date time value is only used by your program, then choose the default system format. The program will work correctly on the system it is running on.

## Send Date Time to Client Apps from web API using JSON

If your program is a web API. And may clients read date time from your API, then you should write in API documentation about the date time format. But, who reads the docs…. There must be some one standard, followed by all, so that life of programmers become easy!!!!

There is a solution, **use the iso standard**.

If your web API is built in ASP.NET or Python or Node. The API will convert the date time format to JSON, which is plain text. The JSON will be read by clients in React, Angular, Android or other system.

## Pass Date Time Values Between .NET Core and React

### Update in ASP.NET Core Web API

In ASP.NET Core web API, you can use Json Options to convert the date time format.

First create a new class DateTimeConverter, which will convert the datetime value to/from string. It is a small class, that just serialize and deserialize JSON data.

Write method will convert DateTime data type to Json String. This method will be used when the web API will convert DateTime to JSON. Here we have used standard iso string format yyyy-MM-ddTHH:mm:ss.

```cs
public class DateTimeConverter : JsonConverter<DateTime>
{
  public override DateTime Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
  {
    return DateTime.Parse(reader.GetString());
  }

  public override void Write(Utf8JsonWriter writer, DateTime value, JsonSerializerOptions options)
  {
    writer.WriteStringValue(value.ToLocalTime().ToString("yyyy-MM-ddTHH:mm:ss"));
  }
}
```

To use this converter, update the Program.cs file

```cs
builder.Services.AddControllers(config =>
{
    // config settings
}).AddJsonOptions(x =>
{
    // use the above date time converter
    x.JsonSerializerOptions.Converters.Add(new DateTimeConverter());
});
```

### Update in React

When the React program calls the web ASP.NET Core web API, it will now get the date in yyyy-MM-ddTHH:mm:ss format. This format is correctly readable in JavaScript’s Date type.

If you are using Formik with Chakra UI, because they both work very well together, you can use datetime-local as input type. Below is an example of Date of Birth text input in React with Chakra UI and Formik.

```react
<FormControl isInvalid={!!errors.dateOfBirth && touched.dateOfBirth}>
  <InputLeftAddon children="Date of Birth" />
  <Field as={Input} id="dateOfBirth" name="dateOfBirth" type="datetime-local" />
  </InputGroup>
  <FormErrorMessage>{errors.dateOfBirth}</FormErrorMessage>
</FormControl>
```

The dateOfBirth field actual value received from the web API will be

```react
dateOfBirth: "2019-09-20T12:50:10"
```

The Chakra UI will parse and display the correct value in the input box. It will also open the default date time picker.

![chakra ui date time](/images/blog/chakra-ui-date-time.jpg "chakra ui date time")

## Conclusion

When your API is running on server, using different time zone and date time format, you can keep the system default format. Don’t use any locale or specific format.

Just convert it to iso or specific format, when you are sending data via API controller. And that too, will be done centrally at one point. If you have 100s of date time fields, you don’t have to change the format at 100s of places. It will be a nightmare to update and standardize them all. Use the AddJsonOptions to convert DateTime to Json.