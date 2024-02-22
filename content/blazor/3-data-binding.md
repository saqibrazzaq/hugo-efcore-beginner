---
title: 3. Data Binding
weight: 3
ShowToc: true
TocOpen: true
---

## One way data binding

Open Counter page in the newly created Blazor project. It has

- A paragraph that should count
- A button that increments count by 1

The count is stored in the variable currentCount. The variable is declared in the code block. Its value is displayed in the HTML paragraph with @currentCount.

The button click event calls the method in the code block. The method increments the same variable.

This is one way data binding, means the variable is updated in code block, and its value can be accessed in HTML.

![one way data binding](/images/blazor/one-way-data-binding.jpg "one way data binding")

## Two way data binding

In one way data binding, the value of variable can be changed from one side e.g. code block.

In two way data binding, the value can be changed from code as well as HTML input.

Add a new form input textbox in the Counter component and set its bind-value to the currentCount variable.

![two way data binding](/images/blazor/two-say-data-binding.jpg "two way data binding")

Now the currentCount variable is also bound to the input textbox. Run the app. You will note that the count value is also displayed in the textbox. Increment the count with button, the textbox value will also be updated. 

Try to change the value in textbox, the variable will also be updated.