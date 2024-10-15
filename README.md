# Introduction

In today’s digital landscape, creating a robust and interactive blogging platform is essential for sharing knowledge and engaging with a wider audience. This article outlines the process of building a developer-friendly blog using Strapi, Next.js, Tailwind CSS, and TypeScript.

**[Strapi](https://strapi.io)** is a leading open-source headless CMS (Content Management System) that provides a flexible and user-friendly interface for managing content. Its scalability and ability to deliver content via a powerful API make it an ideal choice for developers looking to create dynamic applications.

**[Next.js](https://nextjs.org/docs)** is a robust React framework that enhances web applications with features like server-side rendering, static site generation, and automatic code splitting. These capabilities ensure optimal performance and a seamless user experience.

This documentation will guide you through the steps required to set up your blog, covering backend configuration with Strapi, frontend development with Next.js, SEO optimization, pagination, and search functionality. By the end of this tutorial, you will have a comprehensive understanding of building a fully functional, developer-friendly blog. Let’s get started!

# Before Getting Started

Here's the final result of the blog website you will build if you want to see it. Check out the [project repo here.](https://github.com/chrismbah/blog-strapi)

# Prerequisites

- This tutorial uses the latest version of Strapi at the time of writing this article `v5.0.x`

- Node `v18.x.x` or `v20.x.x.` You can download Node.js [here](https://nodejs.org/en/download/package-manager).

# Setting up project folder

Open up your terminal and create a `blog-strapi` folder to store your project files.

```bash
mkdir blog-strapi
```

Navigate into blog-strapi

```bash
cd blog-strapi
```

# Create a Standard Strapi App

Create your Strapi app in a folder named backend.

```bash
npx create-strapi-app@5.0.4 backend --quickstart
```

The `--quickstart` flag sets up your Strapi app with an **SQLite** database and automatically starts your server on port `1337`.

If the server is not already running in your terminal, `cd` into the `backend` folder and run `npm develop` to launch it.

Visit in your browser and register your details in the Strapi Admin Registration Form.`http://localhost:1337/admin`

# Design the content model for a developer friendly blog

Now that your Strapi application is setup and running, fill the form with your personal information to get authenticated to the **Strapi Admin Panel**.

![Strapi signup page](/images/strapi-signup.png "Strapi SignUp")

<!-- ![Create new collection](/images/new-collection-type.png "Create new collection")


![Create blog collection](/images/blog-collection-type.png "Create blog collection") -->

From your admin panel, click on **Content-Type Builder** -> **Create new collection type** tab to create a collection and their field types for your application. In this case we are creating three collections; **Author**, **Blog**, and **Category**.

## Collections Overview

### Blog

The Blog collection will contain the blog posts. It will have the following fields:

**title:** Text (Long Text)
**description:** Text (Short Text)
**content:** Rich Text (Markdown)
**cover:** Media (Image upload)
**slug:** UID (Unique identifier based on the title)
**category:** Relation - many to many (Connect to a Category collection)

### Author

The Author collection will contain the authors of the blog posts. It will have the following fields:

**name:** Text
**avatar:** Media (Image upload)
**email:** Short Text
**blogs:** Relation with the Blogs collection - one to many

### Category

The Category collection will contain the categories of the blog posts. It will have the following fields:
**name:** Text (Short Text)
**slug:** UID
**description:** Text (Short Text)
**blogs:** Relation - many to many

## Understanding Relationships

In Strapi, relationships define how different content types interact with each other.
**One-to-Many Relationship:** This relationship exists when one record in one collection can be associated with multiple records in another collection. For example, one author can have multiple blog posts.
**Many-to-Many Relationship:** This relationship allows multiple records in one collection to be associated with multiple records in another collection. For instance, a blog post can belong to multiple categories and vice versa.

For more detailed information on relationships in Strapi, check out this guide.

## Inputting Dummy Data

Now that you’ve set up your collections, it’s time for some fun! You can input some dummy data for testing purposes. This will help you verify that everything is working smoothly before moving on.

## Allowing Public API Access

Now let's allow API access to your blog to allow you access the blog posts via your API on the frontend. To do that, click on **Settings** -> **Roles** -> **Public**

![Public API ](/images/public-api.png "Public API")

Afterward, collapse the **Blog Permission** tab and mark the **Select all** option and click on the **Save** button.

![Public API ](/images/blog-public-api.png "Blog API")

Scroll downwards and do the same for the **Upload Permission** tab to enable us **upload** images to our strapi backend.

![Public API ](/images/upload-public-api.png "Upload API")

# Create a Standard Next.js App

In a new terminal session, change the directory to `blog-strapi `and run the following command:

```bash
npx create-next-app@latest
```

On installation, you'll see some prompts. Name your project `frontend` and refer to the image below for the other responses.

![Next JS installation](/images/next-install.png "Install Next Js")

Add the following dependencies to your frontend Next app: react-hot-toast, react-icons, react-markdown, react-loader-spinner, remark-gfm, rehype-raw,react-syntax-highlighter for use later.

```bash
cd frontend
npm install react-hot-toast moment react-icons react-markdown react-loader-spinner remark-gfm rehype-raw react-syntax-highlighter
```

These libraries will help you implement features like **notifications**, **date handling**, **icons**, **Markdown rendering** and **loading indicators**. We'd see how it would be implemented later in this article.

# Setup environmental variables

Create a `.env` file in the root of your `frontend` directory and paste the following **environment variables**:

```bash
NEXT_PUBLIC_STRAPI_URL=http://localhost:1337/
NEXT_PUBLIC_PAGE_LIMIT=6
```

The `NEXT_PUBLIC_STRAPI_URL` variable is used to connect to your Strapi backend.
The `NEXT_PUBLIC_PAGE_LIMIT` variable is used to limit the number of blog posts displayed on each
page.

# Setup Types and API Routes

Create a new file called `lib/types.ts` in the `frontend` directory and paste the following

```ts
// export Interface for Image Data
export interface ImageData {
  url: string;
}

// export Interface for Author Data
export interface Author {
  id: number; // Assuming each author has a unique ID
  name: string;
  email: string;
  avatar: ImageData; // Assuming you have an avatar image
}

// export Interface for Category Data
export interface Category {
  documentId: string; // Assuming each category has a unique ID
  name: string;
  description: string; // Optional description
}

export interface BlogPost {
  id: number;
  title: string;
  slug: string;
  description: string;
  content: string; // rich markdown text
  createdAt: string; // ISO date string
  updatedAt: string; // ISO date string
  cover: ImageData; //  featured image
  author: Author; // The author of the blog post
  categories: Category[]; // An array of categories associated with the post
  documentId: string;
}

export interface UserBlogPostData {
  title: string;
  slug: string;
  description: string;
  content: string; //  rich markdown text
}

// Example response structure when fetching posts
export interface BlogPostResponse {
  data: BlogPost[];
}

// Example response structure when fetching a single post
export interface SingleBlogPostResponse {
  data: BlogPost; // The single blog post object
}
```

This shows the structure of the various data types we would recieve for our blog application

Create another file within the `lib` folder called `api.ts` paste the following functions

```ts
import axios, { AxiosInstance } from "axios";
import { UserBlogPostData } from "./types";

export const api: AxiosInstance = axios.create({
  baseURL: `${process.env.NEXT_PUBLIC_STRAPI_URL}`,
});

export const getAllPosts = async (
  page: number = 1,
  searchQuery: string = ""
) => {
  try {
    // If search query exists, filter posts based on title
    const searchFilter = searchQuery
      ? `&filters[title][$containsi]=${searchQuery}`
      : ""; // Search filter with the title
    // Fetch posts with pagination and populate the required fields
    const response = await api.get(
      `api/blogs?populate=*&pagination[page]=${page}&pagination[pageSize]=${process.env.NEXT_PUBLIC_PAGE_LIMIT}${searchFilter}`
    );
    return {
      posts: response.data.data,
      pagination: response.data.meta.pagination, // Return data and include pagination data
    };
  } catch (error) {
    console.error("Error fetching blogs:", error);
    throw new Error("Server error"); // Error handling
  }
};

// Get post by slug
export const getPostBySlug = async (slug: string) => {
  try {
    const response = await api.get(
      `api/blogs?filters[slug]=${slug}&populate=*`
    ); // Fetching single post via the slug parameter
    if (response.data.data.length > 0) {
      // If post exists
      return response.data.data[0]; // Return the post data
    }
    throw new Error("Post not found.");
  } catch (error) {
    console.error("Error fetching post:", error);
    throw new Error("Server error");
  }
};

// Get all posts categories
export const getAllCategories = async () => {
  try {
    const response = await api.get("api/categories"); // Route to fetch Categories data
    return response.data.data; // Return all categories
  } catch (error) {
    console.error("Error fetching post:", error);
    throw new Error("Server error");
  }
};

// Upload image with correct structure for referencing in the blog
export const uploadImage = async (image: File, refId: number) => {
  try {
    const formData = new FormData();
    formData.append("files", image);
    formData.append("ref", "api::blog.blog"); // ref: Strapi content-type name (in this case 'blog')
    formData.append("refId", refId.toString()); // refId: Blog post ID
    formData.append("field", "cover"); // field: Image field name in the blog

    const response = await api.post("api/upload", formData); // Strapi route to upload files and images
    const uploadedImage = response.data[0];
    return uploadedImage; // Return full image metadata
  } catch (err) {
    console.error("Error uploading image:", err);
    throw err;
  }
};

// Create a blog post and handle all fields
export const createPost = async (postData: UserBlogPostData) => {
  try {
    const reqData = { data: { ...postData } }; // Strapi required format to post data
    const response = await api.post("api/blogs", reqData);
    return response.data.data;
  } catch (error) {
    console.error("Error creating post:", error);
    throw new Error("Failed to create post");
  }
};
```

Let's talk about each of the following functions

- `api`: This is an instance of **Axios**, configured with a base URL pointing to your Strapi backend. It simplifies making HTTP requests to your API.

- `getAllPosts(page: number = 1, searchQuery: string = "")`: This asynchronous function fetches all blog posts from the Strapi API, allowing for **pagination** and optional filtering based on a **search query**. It returns the posts and pagination data.

- `getPostBySlug(slug: string)`: This function retrieves a **single** blog post based on its slug. It checks if the post exists and returns the corresponding data or throws an error if not found.

- `getAllCategories()`: This function fetches all categories from the Strapi API, returning their data for use in categorizing blog posts.

- `uploadImage(image: File, refId: number)`: This function handles image uploads to the Strapi API. It prepares a `FormData` object with the image and its associated metadata (like reference ID and field name) and sends it to the server.

- `createPost(postData: UserBlogPostData)`: This function creates a new blog post by sending the provided post data to the Strapi API. It returns the created post's data or throws an error if the creation fails.

# Create Components

## Create the `Navbar` component and implement search handling

Create a `components` folder and create a `Navbar.tsx` file within the folder. The `Navbar` component would handle search functionality as well as links to other routes.

```tsx
"use client";
import React, { useState } from "react";
import Link from "next/link";
import { FaPen, FaSearch, FaTimes } from "react-icons/fa";
import { useSearchParams, usePathname, useRouter } from "next/navigation";

const Navbar = () => {
  const [searchOpen, setSearchOpen] = useState(false);
  const [searchQuery, setSearchQuery] = useState("");
  const searchParams = useSearchParams();
  const pathname = usePathname(); // Get the current pathname
  const { replace } = useRouter(); // Next js function to replace routes

  // Handle search query submission
  const handleSearchSubmit = () => {
    const params = new URLSearchParams(searchParams);
    if (searchQuery) params.set("search", searchQuery);
    else params.delete("search");
    // Always routes with the search query
    replace(`/?${params.toString()}`);
  };

  return (
    <div className="max-w-screen-lg mx-auto sticky top-0 bg-inherit py-3 sm:py-6 z-50  ">
      <nav className="flex justify-between items-center mb-2 p-4 ">
        <div className="flex items-center gap-4">
          <Link href="/">
            <h1 className="font-bold text-xl font-jet-brains text-white">
              DEV.BLOG
            </h1>
          </Link>
          <button
            onClick={() => setSearchOpen((prev) => !prev)}
            className="text-xl text-white hover:text-purple-400 transition-colors"
          >
            {searchOpen ? <FaTimes /> : <FaSearch />}
          </button>

          {/* Search Box (toggles visibility) */}
          {searchOpen && (
            <div className="ml-4 flex items-center gap-2">
              <input
                type="search"
                value={searchQuery}
                onChange={(e) => setSearchQuery(e.target.value)}
                placeholder="Search posts..."
                defaultValue={searchParams.get("search")?.toString()}
                className="bg-gray-800 appearance-none placeholder:text-sm placeholder:font-normal text-sm text-white placeholder-gray-400 border-b-2 border-purple-500 focus:border-purple-300 outline-none px-2 py-1 rounded-md"
              />
              <button
                onClick={handleSearchSubmit}
                className="bg-purple-600 text-sm hover:bg-purple-500 text-white px-2 py-1 rounded-md transition-colors"
              >
                Search
              </button>
            </div>
          )}
        </div>

        <ul className="flex items-center gap-6 font-medium transition-colors font-jet-brains">
          <li
            className={
              pathname === "/"
                ? "text-purple-400"
                : "text-white hover:text-purple-400"
            }
          >
            <Link href="/">Blogs</Link>
          </li>
          <li
            className={
              pathname === "/write"
                ? "text-purple-400"
                : "text-white hover:text-purple-400"
            }
          >
            <Link href="/write">
              <FaPen className="hover:text-purple-400" />
            </Link>
          </li>
        </ul>
      </nav>
    </div>
  );
};

export default Navbar;
```
The `Navbar` component imports necessary components and functions

- The `useState` is used to manage the search bar visibility (`searchOpen`) and search query (`searchQuery`).

- `useSearchParams` helps retrieve the current search parameters, and `usePathname` provides the current route. `useRouter` is used to programmatically update the route when submitting a **search query**.

- When the user types in the search input and clicks the "**Search**" button, the query is captured and appended to the URL as a search parameter. If the search query is empty, the search parameter is removed.

- It also includes navigation links for `Blogs` and `Write` pages, with the active route highlighted by changing text color using pathname.

Create a `Loader` and `Pagination` component within the `components` folder to for the **data handling** and **pagination**


##  Create the `Loader` component

```tsx
// components/Loader.tsx
import { LineWave } from "react-loader-spinner";

const Loader = () => {
  return (
    <LineWave
      visible={true}
      height="150"
      width="150"
      color="#7e22ce"
      ariaLabel="line-wave-loading"
      wrapperStyle={{}}
    />
  );
};

export default Loader;

```
## Create the `Pagination` component
```tsx
import React from "react";
import { FaArrowLeft, FaArrowRight } from "react-icons/fa"; // Import arrow icons from react-icons

interface PaginationProps {
  currentPage: number;
  totalPages: number;
  onPageChange: (newPage: number) => void;
}

const Pagination: React.FC<PaginationProps> = ({
  currentPage,
  totalPages,
  onPageChange,
}) => {
  return (
    <div className="mt-8 flex justify-center items-center space-x-4">
      <button
        disabled={currentPage === 1}
        onClick={() => onPageChange(currentPage - 1)}
        className={`px-4 py-2 bg-purple-700 text-white rounded ${
          currentPage === 1
            ? "opacity-50 cursor-not-allowed"
            : "hover:bg-purple-600"
        }`}
      >
        <FaArrowLeft /> {/* Previous button icon */}
      </button>
      <span className="text-white">
        Page {currentPage} of {totalPages}
      </span>
      <button
        disabled={currentPage === totalPages || currentPage > totalPages}
        onClick={() => onPageChange(currentPage + 1)}
        className={`px-4 py-2 bg-purple-700 text-white rounded ${
          currentPage === totalPages || currentPage > totalPages
            ? "opacity-50 cursor-not-allowed"
            : "hover:bg-purple-600"
        }`}
      >
        <FaArrowRight /> {/* Next button icon */}
      </button>
    </div>
  );
};

export default Pagination;

```

