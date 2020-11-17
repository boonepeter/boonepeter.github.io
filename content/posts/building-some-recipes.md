---
layout: post
title: "Building some-recipes.com"
author: Peter Boone
tags: ["tech", "recipes", "node.js", "react", "typescript", "MongoDB"]
date: 2020-11-16
draft: false
---

## Background

Earlier this year I made my way through [Fullstack Open](https://fullstackopen.com/en/), a course designed to teach modern Javascript-based web development.

In their words:

>The main focus is on building single page applications with ReactJS that use REST APIs built with Node.js. The course also contains a section on GraphQL, a modern alternative to REST APIs. The course covers testing, configuration and environment management, and the use of MongoDB for storing the application’s data.

The course was very practical and had exercises that made sure you really understood the material after reading it. Learning-by-doing is how I learn best, and this course made sure that you were building while learning.

## some-recipes.com

After completing the course, I built [some-recipes.com](https://some-recipes.com/) to apply what I learned. You can view the [source code here](https://github.com/boonepeter/some-recipes). The goal of the website was to be a place where a user can easily __import recipes__ from anywhere on the web and store them in one location. I get annoyed by ads and having to scroll through long stories to get to the actual recipe, so I wanted the UI to be simple and free from clutter.

### Importing recipes

After doing a little digging and trying a few different methods, I realized that schema.org and Google made it fairly easy to extract recipe information from a website.

When you google a recipe, you may see something like this:

![google search results for pizza recipe](/imgs/some-recipes/pizza-search.png)

To get your recipe to show up in searches, Google [recommends](https://developers.google.com/search/docs/data-types/recipe) that you add your recipe in the [schema.org Recipe](https://schema.org/Recipe) format to the `application/ld+json` of the webpage. This is what that information might look like:

```json
 {
     "@context":"http://schema.org",
     "@type":"Recipe",
     "name":"Homemade Pizza",
     "url":"https://www.simplyrecipes.com/recipes/homemade_pizza/",
     "image":"https://www.simplyrecipes.com/wp-content/uploads/2007/01/homemade-pizza-horiz-a-1200.jpg",
     "author":
        {
            "@type":"Person",
            "name":"Elise Bauer",
            "url":"https://www.simplyrecipes.com/author/elise-bauer/"
        },
    "description":"Classic homemade pizza recipe, including pizza dough and toppings, step-by-step instructions with photos.  Make perfect pizza at home!",
    "recipeCuisine":"Italian",
    "keywords":"Pizza, Tomato Sauce",
    "recipeCategory":["Dinner"],
    "recipeInstructions":
        [{
            "@type":"HowToSection",
            "name":"Making the Pizza Dough",
            "position":1,
            "itemListElement":
                [
                    {
                        "@type":"HowToStep",
                        "position":1,
                        "text":"Proof the yeast: Place the warm water in the large bowl of a heavy duty stand mixer. Sprinkle the yeast over the warm water and let it sit for 5 minutes until the yeast is dissolved.\n \nAfter 5 minutes stir if the yeast hasn't dissolved completely. The yeast should begin to foam or bloom, indicating that the yeast is still active and alive.\n(Note that if you are using \"instant yeast\" instead of \"active yeast\", no proofing is required. Just add to the flour in the next step.)"
                    },
                    {
                        "@type":"HowToStep",
                        "position":2,
                        "text":"Make and knead the pizza dough: Add the flour, salt, sugar, and olive oil, and using the mixing paddle attachment, mix on low speed for a minute. Then replace the mixing paddle with the dough hook attachment.\nKnead the pizza dough on low to medium speed using the dough hook about 7-10 minutes.\nIf you don't have a mixer, you can mix the ingredients together and knead them by hand.\nThe dough should be a little sticky, or tacky to the touch. If it's too wet, sprinkle in a little more flour."
                    }
                ]
        }],
    "cookTime":"PT00H30M",
    "prepTime":"PT02H00M",
    "totalTime":"PT2H30M",
    "recipeYield":"Makes 2 10-12-inch pizzas"
}
```

As you can see, when websites adhere to this standard, it makes it quite easy to pull the recipe out.

### Architecture

![some-recipes.com architecture diagram](/imgs/some-recipes/architecture.png)

I built the [frontend](https://github.com/boonepeter/some-recipes/tree/master/frontend) of the website with React + Typescript and styled it with [`react-bootstrap`](https://react-bootstrap.github.io/) and `fontawesome` icons.

#### React impressions

I enjoyed creating small, reusable components and combining them to create the larger UI. I also like how React is somewhat functional. It encourages you to create pure functions and have unidirectional data flow. The `useState()` Hook API was very easy to use and made it explicit whenever I was modifying the state of the component.

While I did not do a lot of custom styling, it does make me uncomfortable mixing `CSS` right in with `JSX` components. I think I will stick with keeping them separate, although it seems like a good number of people encourage you to combine them. I also struggled a bit with `BrowserRouter`. It was easy to start using (setting up `Switch` with `paths`). But when I needed to "manually" change the route (i.e. navigate to a newly created recipe) I had trouble implementing that. I have it working now, but I need to do more reading and exploring to understand how it's meant to be used.

#### Node.js/Express + MongoDB

I built the [API](https://github.com/boonepeter/some-recipes/tree/master/backend) for the website using Node.js and Express (also with Typescript). It was pretty quick to get up and running and then iterate upon. I found the interface between JSON and Javascript objects to be slick (when compared to Python dicts -> JSON for example). It seemed very natural to convert between the two. Mongoose is a pretty nice ORM that interfaces with MongoDB well. The flexibility of the NoSQL database was nice to have while I was changing my schema during development. There are some other advantages (and disadvantages), but that feature is what I appreciated the most.

The API endpoints are what you would expect: create, read, update, and delete users/recipes/lists. The API also has a `parse?url=...` endpoint which pulls the recipe information from a website like I explained above.

I implemented my own user validation with JSON Web Tokens (JWT). If I was really concerned about user data, I would not have implemented as much of it by hand as I did, but it was an interesting learning experience. I'm sure the way I implemented it is not full-proof, so don't upload any extra secret recipes!

### Deployment

I use GitHub Actions to build the frontend, copy it to the backend to serve, and deploy to Azure WebApps.

## Next steps

There are a few things this site needs...we'll see what I get around to.

- __Search__: my current search implementation is broken in a few ways. I want to implement some debouncing and live results as the user types. The UI needs to support searching for tags (the API supports it). Maybe search for both tags and titles
- __Ingredients__: A big feature to implement would be ingredients and recipe scaling. Right now, the ingredients are just lists of strings. If I parsed the ingredients and amounts, then users could search by ingredient and scale recipes up and down (or convert to metric).
- __Users__: I want to improve the profile view. It is pretty basic at this point. User should be able to upload a profile pic. Follow or friend other users.
- __Lists__: Users should be able to create more lists (like Spotify playlists) and add recipes to them. They could then share those.
- __Print view__: since my site is all about simplicity, I should make a simple print view (compact, no picture)
- __Recipe pics__: right now, recipes only have picture if imported from another site, and they are not hosted by me. I need to finish implementing the image upload feature if a user creates their own recipe.
- __Pagination__: since there aren't many recipes on the site at the moment, I don't have to worry about displaying too many recipes on the browse view. I will need to implement pagination at some point.
- __Popular or featured__: maybe sort by popularity?
- __Email__: there is no way for the user to recover their password. Also, emails are case sensitive which needs to change.

## Final thoughts

Getting a minimal viable product that I actually personally use was a nice accomplishment. I start a lot of projects and don't always finish them, so getting something out is good motivation for me to keep working on it. I read [this article](https://www.zainrizvi.io/blog/do-more-by-doing-less/) recently and it resonated with me. Put simply - set time constraints and trim the fat off of your personal projects so you can actually finish something.