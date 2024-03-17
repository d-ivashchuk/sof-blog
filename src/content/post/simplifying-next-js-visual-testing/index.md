---
title: "Simplifying Next.js visual regression testing"
description: "Add visual tests to your Next.js pages in minutes via sitemap"
publishDate: "1 March 2024"
updatedDate: "1 March 2023"
coverImage:
  src: "./cover.png"
  alt: "Blog post wallpaper"
tags: ["visual testing", "automated QA"]
canonical: "https://lost-pixel.com/blog/post/simplifying-next-js-visual-regression-testing"
---

## Status quo

I am currently building [lost-pixel](https://github.com/lost-pixel/lost-pixel), an open-source visual regression testing tool and so far we have tried to make the developer experience click well by natively integrating with [Storybook](https://github.com/storybookjs/storybook), [Ladle](https://github.com/tajo/ladle) & [Histoire](https://github.com/histoire-dev/histoire), - the most prominent component development tools in the industry.

We've recognised right from the start that apart from testing the components, we, as the frontend engineers are also interested in making sure that those components when used to form a cohesive picture on our websites. This is also a huge shame when that cohesiveness breaks and we have a poor UI experience for the users :D Hence, we've also added **pages mode** to lost-pixel which allows users to specify an array of pages to test in a simple but extensible format for any frontend framework(in this example we will be using [Next.js](https://github.com/vercel/next.js))

```javascript
 pages: [{ path: '/app', name: 'app' }, {path:'/pricing', name: 'pricing'}],
```

I started using it myself extensively on our landing page, inside of our SaaS platform, and everywhere where visual tests were making sense. But it always felt **like something was missing**, as there was still a bunch of manual work for engineers to do to create those lists of pages.

We came up with another abstraction over it! Allowing users to come up with their way of informing us about what they want to visually test in their application:

```javascript
pagesJsonUrl: './lost-pixel-pages.json',
```

With **pagesJsonUrl** users got the option to generate the list of pages in the format that we expect(thank you typescript for making it safe) and just point us to the file or URL where this JSON is available for us to consume.

Yet, while being super useful in projects like [Gatsby](https://github.com/gatsbyjs/gatsby) where there is a way of nicely getting the list of the website pages from their internal API it **was still too big of a hustle** for projects where outputting the pages was not so straightforward.

There came my good friend, Oleg, who is also the founder of [Webstudio](https://github.com/webstudio-is/webstudio), an open-source Webflow alternative and he suggested something that flipped the game for us by adding a simple feature that took **less than an hour to write**!

## Solution

As with everything web platform-related, given the plethora of options out there it is very easy to overlook some things that make sense. I felt stupid that I overlooked it and in retrospect, this feature was crucial to add at the same time we introduced the **pagesJsonUrl**

Enter `npx lost-pixel page-sitemap-gen` 🎉

Most modern websites have **sitemaps** that are dynamically generated which is an easy fit for next.js websites and those sitemaps usually have the list of the most important pages that you care about on your website.

By reading the sitemap, parsing it & creating the list of pages to be consumed by the pagesJsonUrl I finally closed the circle! Let me show you an example of a simple GitHub action:

```yaml
      ...

      - name: Build next
        run: pnpm run build
      - name: Start next
        run: pnpm run start &

      - name: Generate Sitemap
        run: npx lost-pixel page-sitemap-gen http://172.17.0.1:3000/sitemap.xml "./lost-pixel-pages.json"

      - name: Lost Pixel
        uses: lost-pixel/lost-pixel@v3.10.0
        env:
          LOST_PIXEL_API_KEY: ${{ secrets.LOST_PIXEL_API_KEY }}
```

```typescript
export const config: CustomProjectConfig = {
	pageShots: {
		pagesJsonUrl: "./lost-pixel-pages.json",
		baseUrl: "http://172.17.0.1:3000",
	},
	lostPixelProjectId: "clb1xlaeo1299101qqkt94cn43",
	apiKey: process.env.LOST_PIXEL_API_KEY,
};
```

What happens here is simple, but yet very powerful for the maintainability & simplicity of your visual tests.

1. We build the Next.js app(the sitemap is dynamically generated so we always have a fresh version of it)

2. We run `npx lost-pixel page-sitemap-gen` to take the fresh version of the sitemap and generate the array of pages for lost-pixel to snapshot

3. We run lost-pixel over that array

Suddenly instead of manually adding the page to the array every time we add something new it is fully automated. The best thing about it all - it can be recreated for **any website that has a sitemap**, either dynamic or static 🚀
Here I've added 50 pages to our visual testing suite for our landing page in less than 5 minutes. [Lost Pixel Platform](https://www.lost-pixel.com/) can better showcase it but you can also use it through our open-source lost-pixel and easily achieve the same functionality without spending a dime!

![](https://cdn.sanity.io/images/809vdkvp/production/99fa2cdca9c5cf547faa197426a30ebba57d702a-1738x828.png)

## Ending thoughts

I am pretty excited about this little feature as it unlocks a new level of developer experience improvement within lost-pixel. I would be excited to hear your thoughts and see if I have missed some other awesome use cases with this feature 😀
