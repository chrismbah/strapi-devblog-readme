# Introduction

In today’s digital landscape, creating a robust and interactive blogging platform is essential for sharing knowledge and engaging with a wider audience. This article outlines the process of building a developer-friendly blog using Strapi, Next.js, Tailwind CSS, and TypeScript.

**[Strapi](https://strapi.io)** is a leading open-source headless CMS (Content Management System) that provides a flexible and user-friendly interface for managing content. Its scalability and ability to deliver content via a powerful API make it an ideal choice for developers looking to create dynamic applications.


**[Next.js](https://nextjs.org/docs)** is a robust React framework that enhances web applications with features like server-side rendering, static site generation, and automatic code splitting. These capabilities ensure optimal performance and a seamless user experience.


<!-- **[Tailwind CSS](https://tailwindcss.com/)** is a utility-first CSS framework that allows for rapid and responsive design implementation, enabling developers to create aesthetically pleasing interfaces without the complexity of custom styles. -->


This documentation will guide you through the steps required to set up your blog, covering backend configuration with Strapi, frontend development with Next.js, SEO optimization, pagination, and search functionality. By the end of this tutorial, you will have a comprehensive understanding of building a fully functional, developer-friendly blog. Let’s get started!

# Before Getting Started

Here's the final result of the blog website you will build if you want to see it. Check out the [project repo here.](https://github.com/chrismbah/strapi-dev-blog)


# Prerequisites

- This tutorial uses the latest version of Strapi at the time of writing this article ```v5.0.x```

- Node ```v18.x.x``` or ```v20.x.x.```  You can download Node.js [here](https://nodejs.org/en/download/package-manager).



# Setting up project folder

Open up your terminal and create a ```strapi-dev-blog``` folder to store your project files.


```bash
mkdir strapi-dev-blog
```


Navigate into strapi-dev-blog

```bash
cd strapi-dev-blog
```

# Create a Standard Strapi App

Create your Strapi app in a folder named backend.


```bash
npx create-strapi-app@5.0.4 backend --quickstart
```


The ```--quickstart``` flag sets up your Strapi app with an SQLite database and automatically starts your server on port ```1337```.


If the server is not already running in your terminal, ```cd``` into the backend folder and run ```npm develop``` to launch it. 

Visit  in your browser and register your details in the Strapi Admin Registration Form.```http://localhost:1337/admin```

# Design the content model for a developer friendly blog

Now that your Strapi application is setup and running, fill the form with your personal information to get authenticated to the **Strapi Admin Panel**.

![Strapi signup page](/images/strapi-signup.png "Strapi SignUp")

From your admin panel, click on the **Content-Type Builder** -> **Create new collection type** tab to create a **Blog** collection for your application.

![Create new collection](/images/new-collection-type.png "Create new collection")

A modal will appear, containing forms for your collection name. Enter "Blog" as the display name which would be the name of the collection. Strapi automatically fills in the singular and plural api ids, then click on **Continue** to proceed.

![Create blog collection](/images/blog-collection-type.png "Create blog collection")

Next, select the **fields** for your collection. For this blog application, choose the following:

- **Title:** Text (Long Text)
- **Description:** Text (Short Text)
- **Content:** Rich Text (Markdown)
- **Cover:** Media (Image upload)
- **Slug:** UID (Unique identifier based on the title)
- **Category:** Relation (Connect to a Category collection)

