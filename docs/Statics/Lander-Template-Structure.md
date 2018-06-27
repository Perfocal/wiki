# Lander Structure

Lander structure is as follows:

`Templates --< Landers --< Variables` | where `--<` is a **one-to-many** relationship.

This structure ensures that all types of `Landers` are split up into their various parts, i.e. `Templates`. The `Landers` are there to create different URLs for users to access. These pages are then built into their own folders within the `public/` folder where the naming of pages is `public/[Lander]/index.html`.

Templates are stored in their own folder within `views/` called `templates/`. This is because the templates are full pages that  will be reused more than the single-page views, which is why they are not part of the `parts/` subfolder.

## Variables used in lander templates. 

| Variable Name           | Type     |
| ----------------------- | -------- |
| title                   | string   |
| metaDescription         | string   |
| metaKeywords            | string[] |
| mainCover               | string   |
| headLine                | string   |
| headLineSubtitle        | string   |
| headLineCTA             | string   |
| sectionTitle1           | string   |
| sectionSubtitle1        | string   |
| sectionTitle2 (optional)| string   |
| sectionSubtitle2        | string   |
| paragraphDescription    | string   |
| sectionTitle3 (optional)| string   |
| sectionSubtitle3        | string   |
| sectionTitle4 (optional)| string   |
| sectionSubtitle4        | string   |
| typedTexts              | string[] |
All variables used here are injected into the `.ejs` file via `<%- %>` tags which output **unescaped** values into the template. This means that it is possible to use `HTML` tags in the variable which can then be displayed in resulting build file. An example of such a use case is in the `headLine` where we have used `text <b>bold text</b>`

## Variables used in join lander templates.

| Variable Name           | Type     |
| ----------------------- | -------- |
| title                   | string   |
| metaDescription         | string   |
| metaKeywords            | string[] |
| headLine                | string   |
| headLineSubtitle        | string   |


## Variables used in `.ejs` parts

**Please note: These variables are guidelines on how these parts will be structured. However, due to issues with webpack and EJS, these parts will be hard-coded into each template separately for the forseeable future until a solution is found.**

### Gallery
| Variable Name | Type                                                         |
| ------------- | ------------------------------------------------------------ |
| galleryImages | { hrefSm: string, hrefStd: string, galleryName: string, portrait: boolean, alt: string}[] |

### Testimonials
| Variable Name   | Type                                                         |
| --------------- | ------------------------------------------------------------ |
| testimonialList | { images: string[], reviewBody: string, avatarUrl: string, name: string, title: string }[] |

### Pricing
| Variable Name | type                                                         |
| ------------- | ------------------------------------------------------------ |
| pricingList   | { packageName: string, price: number, duration: {number: string, word: string}, noOfPhotos: number, mostPopular: boolean, package: {sceneId: number, packageId: number} }[] |

## Lander template sample (May 2018)

The example below displays the structure and variables of a template. This was used as a reference when setting up the initial template and variable structure.

![](https://user-images.githubusercontent.com/8979275/41170076-d4354866-6b42-11e8-86be-2e945ff787f7.jpg)