---
title: "django-calculation, Make simple calculations in django forms."
layout: post
date: 2022-04-11 12:00
tag: django
image: false
headerImage: false
projects: true
hidden: true # don't count this post in blog pagination
description: "The app provide a Django Widget that derives its value from a expression defined in the widget instance."
category: project
author: blasferna
externalLink: false
---


Make simple calculations in your Django forms using django-calculation. [django-calculation](https://github.com/blasferna/django-calculation). 

## How does it work?

The application offers a series of Widgets that allow you to specify mathematical expressions and/or calls to JavaScript functions.

A basic example would be performing a multiplication to calculate the total price of a sales record.



```python
amount = forms.DecimalField( 
    widget=calculation.FormulaInput('quantity*price')  
) 
```

At runtime, the expression `quantity*price` is replaced by the values ​​corresponding to the fields of the same name present in the form.

It currently supports formula-type expressions, as well as addition operations, average calculations, finding the minimum value, maximum value, and record counting.


## Motivation

To be honest, I'm too lazy to write JavaScript code in templates that have forms that require some kind of mathematical calculation. In a medium or large project the need can be several hundred or thousands of lines of code.

As I don't have the need to implement advanced frontend libraries, such as React, Vue or other available since Django's MTV adapts quite well to the projects I participate in. Looking for a way to avoid writing JavaScript code for those situations, I decided to try something similar to what Salesforce offers with FormulaField, but in this case something much more practical focused on what I needed.

For several years I have been a user of Oracle Forms 6i, which had something very practical: Indicate that the content of a text box be generated from the execution of a formula and something very interesting happened when you referenced another text box of the same type , the execution respected the dependency of the content of the formula, that is, it was executed in cascade.

With all that in mind I started the development.

## Development process

The idea is quite simple, in theory, to execute the expressions indicated in the definition of the text field in a Django form, respecting the references they may have in other expressions of the same context. Activate them when the source fields are modified.

### Functional flow

After thinking about the flow for several days, I came to the conclusion that it could work as follows.

1. Find Formulated Fields

The first thing would be to identify the fields that contain formulas, I decided to apply the search according to the `data-calculation` attribute.

2. Find dependencies

Then, find all the fields referenced in the formulas, to do this he had to go through all the formulated fields and analyze each of the formulas.

3. Determine the order of execution

To determine the execution order I had to use an algorithm that consists of giving a weight to each formulated field based on the number of times it was referenced, the more times it is referenced, the greater its weight would be and therefore its execution would be after considered fields lighter.

Code used to calculate the weight of the fields

```javascript
function calculateWeight(obj, weight = 0) {
    weight++;
    for (let index = 0; index < obj.dependencies.length; index++) {
        let o = obj.dependencies[index];
        weight = calculateWeight(o, weight);
    }
    return weight;
}
```

Code to order the execution

```javascript
function sortExecution() {
    for (let index = 0; index < calculatedFields.length; index++) {
        let obj = calculatedFields[index];
        obj.weight = calculateWeight(obj);
    }
    calculatedFields.sort(function (a, b) {
        return a.weight - b.weight;
    });
}
```

4. Find source fields

Then I had to find those fields that are going to execute the calculations, that is, the source fields, those that, when they undergo changes, will execute the calculations in the places where they have been referenced.

5. Add events

Having the source fields, all that remains is to add the event that would trigger the executions, I decided to use the `blur` event because it is triggered after losing the focus.


## Installation

```bash
pip install django-calculation 
```

Add `calculation` in `INSTALLED_APPS`

```python
INSTALLED_APPS = [
    ...
    'calculation',
]
```

## Usage

Import `calculation` and complete the definition.

### Example

Using `FormulaInput` widget

```python
from django import forms

import calculation


class TestForm(forms.Form):
    quantity = forms.DecimalField()
    price = forms.DecimalField()
    amount = forms.DecimalField(
        widget=calculation.FormulaInput('quantity*price') # <- using single math expression
    )
    apply_taxes = forms.BooleanField(initial=True)
    tax = forms.DecimalField(
        # using math expression and javascript functions.
        widget=calculation.FormulaInput('apply_taxes ? parseFloat(amount/11).toFixed(2) : 0.0') 
    )
```

`django-calculation` works with static files and therefore it is necessary to include the media of the form in the template file.

```django
<form method="post">
    {% csrf_token %}
    {{ form }}
    <input type="submit" value="Submit">
</form>

{{ form.media }}
```

In action:


![calculation](https://user-images.githubusercontent.com/8385910/142947517-49a5d6a0-6a6c-41d6-8f14-a140ad44fa1e.gif)


## Open Source

I was using it for several weeks adjusting some details and after a while I decided to release the project with the hope that it will be useful for other people since for me it is very practical.

For that I had to apply some improvements such as including it in the Python PyPI package manager.


### Repercussion

It wasn't long after the first release that I received some emails from users asking about some details of the library, that was exciting.

### Contribute

If you have any ideas on how to improve the library or found some bugs, feel free to open an issue at [https://github.com/blasferna/django-calculation/issues](https://github.com/blasferna/django-calculation/issues). 

