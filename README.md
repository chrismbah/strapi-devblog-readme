# Introduction

Content is in high demand today, and a good blogging site is a great way of reaching out to people, for learning, educative or entertainment purposes. In this article, we shall describe how to go about integrating the Strapi framework with Next.js that is optimized with Tailwind CSS to develop a blog that targets developers.

**[Strapi](https://strapi.io)** is an open source headless content management system **(CMS)** that allows users to build and manage content easily without having to worry about how to deliver it through a frontend application. This is quite advantageous to developers who want to build dynamic applications as it has an API that can be used to render the applications very fast.

**[Next.js](https://nextjs.org/docs)** is an all purpose react framework providing features such as server-side rendering, static site generation, automatic code splitting among many other web application enhancement features. These features are essential in ensuring that the application is performing at its best and the user is getting a good experience.

This guide will take you through what it takes to setup up your blog from backend with strapi, develop your frontend using next js focus on SEO, add pagination and search. By the end of this tutorial, you should know how to create a complete and easy to use blog. Time to begin!

# Before Getting Started

Here's the final result of the blog website you will build if you want to see it. Check out the [project repo here.](https://github.com/chrismbah/blog-strapi)

# Prerequisites

- This tutorial uses the latest version of Strapi at the time of writing this article `v5.0.x`

- Node version `v18.x.x` or `v20.x.x.` You can download Node.js [here](https://nodejs.org/en/download/package-manager).

# Setting up project folder

Open up your terminal and create a `blog-strapi` folder to store your project files.

```bash
mkdir blog-strapi
```

Navigate into the `blog-strapi` directory

```bash
cd blog-strapi
```

# Create a Standard Strapi App

Create your Strapi app in a folder named `backend`.

```bash
npx create-strapi-app@5.0.4 backend --quickstart
```

The `--quickstart` flag sets up your Strapi app with an **SQLite** database and automatically starts your server on port `1337`.

If the server is not already running in your terminal, `cd` into the `backend` folder and run `npm develop` to launch it.

Visit `http://localhost:1337/admin` in your browser and register in the **Strapi Admin Registration Form**.

# Design the content model for a developer friendly blog

Now that your Strapi application is setup and running, fill the form with your personal information to get authenticated to the **Strapi Admin Panel**.

![Strapi signup page](./images/strapi-signup.png "Strapi SignUp")

<!-- ![Create new collection](./images/new-collection-type.png "Create new collection")


![Create blog collection](./images/blog-collection-type.png "Create blog collection") -->

From your admin panel, click on **Content-Type Builder** -> **Create new collection type** tab to create a collection and their field types for your application. In this case we are creating three collections; **Author**, **Blog**, and **Category**.

## Collections Overview

### Blog

The Blog collection will contain the blog posts. It will have the following fields:

- **title:** Text (Long Text)
- **description:** Text (Short Text)
- **content:** Rich Text (Markdown)
- **cover:** Media (Image upload)
- **slug:** UID (Unique identifier based on the title)
- **category:** Relation - many to many (Connect to a Category collection)

### Author

The Author collection will contain the authors of the blog posts. It will have the following fields:

- **name:** Text
- **avatar:** Media (Image upload)
- **email:** Short Text
- **blogs:** Relation with the Blogs collection - one to many

### Category

The Category collection will contain the categories of the blog posts. It will have the following fields:
- **name:** Text (Short Text)
- **slug:** UID
- **description:** Text (Short Text)
- **blogs:** Relation - many to many

## Understanding Relationships

In Strapi, relationships define how different content types interact with each other.

- **One-to-Many Relationship:** This relationship exists when one record in one collection can be associated with multiple records in another collection. For example, one author can have multiple blog posts.

- **Many-to-Many Relationship:** This relationship allows multiple records in one collection to be associated with multiple records in another collection. For instance, a blog post can belong to multiple categories and vice versa.

For more detailed information on relationships in Strapi, check out this [guide](https://strapi.io/blog/understanding-and-using-relations-in-strapi).

## Inputting Dummy Data

Now that you’ve set up your collections, it’s time for some fun! You can input some dummy data for testing purposes. This will help you verify that everything is working smoothly before moving on.

## Allowing Public API Access

Now let's allow API access to your blog to allow you access the blog posts via your API on the frontend. To do that, click on **Settings** -> **Roles** -> **Public**

![Public API ](./images/public-api.png "Public API")

Afterward, collapse the **Blog Permission** tab and mark the **Select all** option and click on the **Save** button.

![Public API ](./images/blog-public-api.png "Blog API")

Scroll downwards and do the same for the **Upload Permission** tab to enable us **upload** images to our strapi backend.

![Public API ](./images/upload-public-api.png "Upload API")

# Create a Standard Next.js App

Strapi supports multiple frontend frameworks including **React**, **Next.js**, **Gatsby**, **Svelte**, **Vue Js** etc. but for this application we'd be making use of **Next Js**.


In a new terminal session, change the directory to `blog-strapi `and run the following command:

```bash
npx create-next-app@latest
```

On installation, you'll see some prompts. Name your project `frontend` and refer to the image below for the other responses.

![Next JS installation](./images/next-install.png "Install Next Js")

**Note:** Make sure you select the recommended `App Router` for Next JS because that's what we'll be using throughout our application
## Install necessary dependencies

Add the following dependencies to your frontend Next app: `react-hot-toast`, `react-icons`, `react-markdown`, `react-slugify`, `@uiw/react-markdown-editor`, `react-loader-spinner`,`remark-gfm`, `rehype-raw`, `react-syntax-highlighter`, `@tailwindcss/typography`  for use later.

```bash
cd frontend
npm install react-hot-toast @tailwindcss/typography moment react-icons react-markdown react-loader-spinner remark-gfm rehype-raw react-syntax-highlighter
```
These libraries would be used throughout the application serving several purposes.

- **`react-hot-toast`**: Provides easy-to-use notifications for user interactions.
- **`@tailwindcss/typography`**: Enhances the typography styles in your application for better readability.
- **`moment`**: A library for parsing, validating, manipulating, and displaying dates and times.
- **`react-icons`**: Offers a comprehensive set of icons for use throughout your application.
- **`react-markdown`**: Renders Markdown content as React components, enabling rich text display.
- **`react-loader-spinner`**: Displays loading indicators to enhance user experience during asynchronous operations.
- **`remark-gfm`**: Enables GitHub Flavored Markdown features, such as tables and strikethroughs, in your Markdown content.
- **`rehype-raw`**: Allows rendering of raw HTML in Markdown, giving flexibility in content formatting.
- **`react-syntax-highlighter`**: Provides syntax highlighting for code snippets in your Markdown content.

After installation, add this plugin to your `tailwind.config.ts` file to enable smooth markdown render in your application.

```ts
// tailwind.config.ts
module.exports = {
  theme: {
    // ...
  },
  plugins: [
    require('@tailwindcss/typography'),
    // ...
  ],
}

```
This is necessary when rendering markdown syntax with tailwind in our page.

# Setup environmental variables

Create a `.env` file in the root of your `frontend` directory and add the following **environment variables**:

```bash
NEXT_PUBLIC_STRAPI_URL=http://localhost:1337/
NEXT_PUBLIC_PAGE_LIMIT=6
```

The `NEXT_PUBLIC_STRAPI_URL` variable is used to connect to your Strapi backend.
The `NEXT_PUBLIC_PAGE_LIMIT` variable is used to limit the number of blog posts displayed on each
page for the pagination functionality.

# Setup Types and API Routes

Create a new file called `lib/types.ts` in the `frontend` directory and paste the following

```ts
// lib/types.ts
// export Interface for Image Data
export interface ImageData {
  url: string;
}

// export Interface for Author Data
export interface Author {
  id: number; // Assuming each author has a unique ID
  name: string;
  email: string;
  avatar: ImageData; // Assuming the author has
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
  cover: ImageData; // Assuming this is the structure of your featured image
  author: Author; // The author of the blog post
  categories: Category[]; // An array of categories associated with the post
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
// lib/api.ts
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
    ); // Fetch a single blog post using the slug parameter
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

- `getAllPosts(page: number = 1, searchQuery: string = "")`: This fetches all blog posts from the Strapi API, allowing for **pagination** and optional filtering based on a **search query**. It returns the posts and pagination data.

- `getPostBySlug(slug: string)`: This retrieves a **single** blog post based on its slug. It checks if the post exists and returns the corresponding data or throws an error if not found.

- `getAllCategories()`: This fetches all categories from the Strapi API, returning their data for use in categorizing blog posts.

- `uploadImage(image: File, refId: number)`: This handles image uploads to the Strapi API. It prepares a `FormData` object with the image and its associated metadata (like reference ID and field name) and sends it to the server.

- `createPost(postData: UserBlogPostData)`: This creates a new blog post by sending the provided post data to the Strapi API. It returns the created post's data or throws an error if the creation fails.

# Creating Components

## Creating the `Navbar` component and implement search handling

Create a `components` folder within the `src` folder and create a `Navbar.tsx` file within the folder. The `Navbar` component would handle search functionality as well as links to other routes.

```tsx
// src/components/Navbar.tsx
"use client";
import React, { useState } from "react";
import Link from "next/link";
import { FaPen, FaSearch, FaTimes } from "react-icons/fa";
import { useSearchParams, usePathname, useRouter } from "next/navigation";

const Navbar = () => {
  const [searchOpen, setSearchOpen] = useState(false);
  const [searchQuery, setSearchQuery] = useState("");
  const searchParams = useSearchParams();
  const pathname = usePathname(); // Get the current route path
  const { replace } = useRouter(); // Next js function to replace routes

  // Handle search query submission
  const handleSearchSubmit = () => {
    const params = new URLSearchParams(searchParams);
    if (searchQuery) params.set("search", searchQuery);
    else params.delete("search");
    // Always routes with the search query
    replace(`/?${params.toString()}`);
    setSearchOpen(false); // Close search bar after submission
  };

  return (
    <div className="max-w-screen-lg mx-auto sticky top-0 bg-inherit py-3 sm:py-6 z-50  ">
      <nav className="flex justify-between items-center mb-2 p-4 ">
        <div className="flex items-center gap-4">
          <Link href="/">
            <h1 className="font-bold text-xl text-purple-600 font-jet-brains">
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

## Creating the `Loader` component

The `Loader` component would be used to indicate the loading state when fetching data from strapi

```tsx
// // src/components/Loader.tsx
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

## Creating the `Pagination` component

The `Pagination` component would be used to implement pagination for the blogs

```tsx
// src/components/Pagination.tsx
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

# Setup App Layout

In the `app/layout.tsx` import the `Navbar` component from your `components` folder and `Toaster` component from the `react-hot-toast` package and add them to the layout.

Update the `metadata` object by adjusting the **title**, **description**, and **icon** to suit your application.


```tsx
// app/layout.tsx
import type { Metadata } from "next";
import Navbar from "@/components/Navbar";
import localFont from "next/font/local";
import { Toaster } from "react-hot-toast";
import "./globals.css";

const geistSans = localFont({
  src: "./fonts/GeistVF.woff",
  variable: "--font-geist-sans",
  weight: "100 900",
});
const geistMono = localFont({
  src: "./fonts/GeistMonoVF.woff",
  variable: "--font-geist-mono",
  weight: "100 900",
});

export const metadata: Metadata = {
  title: "DEV.BLOG",
  description:
    "Your go-to resource for all things Strapi—explore best practices, tips, and community insights to elevate your projects",
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en" data-color-mode="dark">
      <body
        className={`${geistSans.variable} ${geistMono.variable} antialiased`}
      >
        <Navbar />
        {children}
        <Toaster />
      </body>
    </html>
  );
}
```
# Creating Pages

## Creating the Home Page

You will create the home page for your blog frontend in this step. In this case, the home page will display a list of all the blog posts.

There should be a file named page.tsx in the `src/app/` directory.

Add the following code to `page.tsx`:

```tsx
// src/app/page.tsx
/* eslint-disable @next/next/no-img-element */
"use client";
import { useEffect, useState } from "react";
import { useSearchParams, useRouter } from "next/navigation";
import Link from "next/link";
import { getAllPosts } from "../lib/api";
import { BlogPost } from "@/lib/types";
import Loader from "@/components/Loader";
import Pagination from "@/components/Pagination";

export default function Home() {
  const [posts, setPosts] = useState<BlogPost[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [totalPages, setTotalPages] = useState(1); // Track total number of pages

  const searchParams = useSearchParams();
  const router = useRouter();

  // Get the search query and page from the URL params
  const searchQuery = searchParams.get("search") ?? "";
  const pageParam = searchParams.get("page");
  const currentPage = pageParam ? parseInt(pageParam) : 1; // Default to page 1 if not present

  useEffect(() => {
    const fetchPosts = async (page: number) => {
      try {
        const { posts, pagination } = await getAllPosts(page, searchQuery);
        setPosts(posts);
        setTotalPages(pagination.pageCount); // Set total pages
      } catch (error) {
        setError("Error fetching posts.");
        console.error("Error fetching posts:", error);
      } finally {
        setLoading(false);
      }
    };

    fetchPosts(currentPage);
  }, [currentPage, searchQuery]); // Re-fetch when page or search query changes

  const handlePageChange = (newPage: number) => {
    // Update the page parameter in the URL
    const newParams = new URLSearchParams(searchParams.toString());
    newParams.set("page", newPage.toString());
    router.push(`?${newParams.toString()}`);
    setLoading(true); // Show loader while fetching
  };

  return (
    <div className="max-w-screen-lg mx-auto p-4">
      {loading && (
        <div className="w-full flex items-center justify-center">
          <Loader />
        </div>
      )}
      {error && <p>{error}</p>}

      {!loading && !error && (
        <>
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            {posts.length > 0 ? (
              posts.map((post) => (
                <div
                  key={post.id}
                  className="cursor-pointer bg-gray-900 rounded-lg overflow-hidden shadow-md hover:shadow-lg transition-shadow"
                >
                  <Link href={`/blogs/${post.slug}`} className="block">
                    {post.cover?.url && (
                      <div className="relative h-36 w-full">
                        <img
                          src={`${process.env.NEXT_PUBLIC_STRAPI_URL}${post.cover.url}`}
                          alt={post.title}
                          className="w-full h-full object-cover"
                        />
                      </div>
                    )}
                    <div className="p-4">
                      <h2 className="text-lg font-semibold font-jet-brains text-white line-clamp-2">
                        {post.title}
                      </h2>
                      <p className="text-gray-400 mt-2 text-sm leading-6 line-clamp-3">
                        {post.description}
                      </p>
                      <p className="text-purple-400 text-sm mt-4 inline-block font-medium hover:underline">
                        Read More
                      </p>
                    </div>
                  </Link>
                </div>
              ))
            ) : (
              <p className="text-gray-400">No posts available at the moment.</p>
            )}
          </div>

          {/* Pagination Controls */}
          <Pagination
            currentPage={currentPage}
            totalPages={totalPages}
            onPageChange={handlePageChange} // Update page when pagination changes
          />
        </>
      )}
    </div>
  );
}
```

This code sets up a Next.js page that will fetch a list of all the blogs from the Strapi API `/blogs` endpoint and renders them in a neat blog-like format.

### Key Points:

- We use the `useEffect` hook to fetch a list of all the blogs from the endpoint and renders them in a neat blog-like format. The `fetchPosts` function uses the `getAllPosts` function in our `/lib/api.tsx` file to make the API request.

- While the data is being fetched, a `Loader` component is displayed. If an error occurs or the post is not found, appropriate messages are shown.

- The `handlePageChange` function is used to manage pagination in our blog. When a user clicks to navigate to a new page, this function updates the URL with the new page number while preserving any existing search parameters. 

- It creates a `new URLSearchParams` object, sets the page parameter to the selected page number, and then uses `router.push` to navigate to the updated URL. Finally, it sets a loading state to true, which can be used to show a loader while the new content is being fetched.

- In each post we are making use of Next Js `Link` component to route users to a single post with each post's unique `slug`. In our next step, we would use the slug to make a query to fetch the data in our single blog post page. 

## Creating the Single Blog page

To create a single blog page, the next step is to set up the necessary folder structure. In the `app` directory, create a new folder named `blogs`, and then create a subfolder called `[slug]` within it. Finally, add a file named `pages.tsx` inside the `[slug]` folder. This structure will look like this: `app/blogs/[slug]/pages.tsx.`

![dynamic routing](./images/dynamic-route.png "Dynamic Routing NextJs")

The reason for creating a folder named `[slug]` is that it allows us to define [dynamic routes](https://nextjs.org/docs/pages/building-your-application/routing/dynamic-routes) in Next.js.

The `[slug]` part of the folder name acts as a placeholder for the unique identifier of each blog post, enabling the application to render different content based on the specific slug in the URL. This way, when users navigate to a blog post, they will see the corresponding content based on its unique slug. For example `/blogs/slug-of-blog-post`

Paste the following code in your `page.tsx` file.

```tsx
// app/blogs/[slug]/page.tsx.
"use client";
import { useEffect, useState } from "react";
import { getPostBySlug } from "../../../lib/api"; // Import your API function
import { useRouter } from "next/navigation";
import { BlogPost } from "@/lib/types";
import Markdown from "react-markdown";
import remarkGfm from "remark-gfm";
import rehypeRaw from "rehype-raw";
import { Prism as SyntaxHighlighter } from "react-syntax-highlighter";
import { dracula } from "react-syntax-highlighter/dist/cjs/styles/prism";
import { FaClipboard } from "react-icons/fa"; // Import your chosen icon
import Loader from "@/components/Loader";
import moment from "moment";
import { toast } from "react-hot-toast";


const handleCopyCode = async (code: string) => {
  try {
    await navigator.clipboard.writeText(code);
    toast.success("Code copied to clipboard!"); // Show toast on error
  } catch (err) {
    console.error("Failed to copy code: ", err);
  }
};

const BlogPostPage = ({ params }: { params: { slug: string } }) => {
  const { slug } = params;
  const [post, setPost] = useState<BlogPost | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);
  const router = useRouter();

  useEffect(() => {
    const fetchPost = async () => {
      if (slug) {
        try {
          // Fetch the post using the slug
          const fetchedPost = await getPostBySlug(slug);
          setPost(fetchedPost);
        } catch (err) {
          setError("Error fetching post.");
          console.log(err);
        } finally {
          setLoading(false);
        }
      }
    };

    fetchPost();
  }, [slug]);

  if (loading)
    return (
      <div className="max-w-screen-md mx-auto flex items-center justify-center">
        <Loader />
      </div>
    );
  if (error) return <p className="max-w-screen-md mx-auto">Error: {error}</p>;
  if (!post) return <p className="max-w-screen-md mx-auto">No post found.</p>;
  console.log(post);
  return (
    <div className="max-w-screen-md mx-auto p-4">
      <h1 className="text-4xl leading-[60px] capitalize text-center font-bold text-purple-800 font-jet-brains">
        {post.title}
      </h1>
      <div className="w-full flex items-center justify-center font-light">
        Published: {moment(post.createdAt).fromNow()}
      </div>

      {/* Categories Section */}
      {post.categories && post.categories.length > 0 && (
        <div className="flex flex-wrap space-x-2 my-4">
          {post.categories.map(({ name, documentId }) => (
            <span
              key={documentId}
              className="border border-purple-900 font-medium px-2 py-2 text-sm"
            >
              {name}
            </span>
          ))}
        </div>
      )}

      {post.cover && (
        <div className="relative h-72 w-full my-4">
          <img
            src={`${process.env.NEXT_PUBLIC_STRAPI_URL}${post.cover.url}`}
            alt={post.title}
            className="rounded-lg w-full h-full object-cover"
          />
        </div>
      )}
      <p className="text-gray-300 leading-[32px] tracking-wide italic mt-2 mb-6">
        {post.description}
      </p>
      <Markdown
        className={"leading-[40px] max-w-screen-lg prose prose-invert"}
        remarkPlugins={[remarkGfm]}
        rehypePlugins={[rehypeRaw]}
        components={{
          code({ inline, className, children, ...props }: any) {
            const match = /language-(\w+)/.exec(className || "");
            const codeString = String(children).replace(/\n$/, "");

            return !inline && match ? (
              <div className="relative">
                <button
                  onClick={() => handleCopyCode(codeString)}
                  className="absolute top-2 right-2 bg-gray-700 text-white p-1 rounded-md hover:bg-gray-600"
                  title="Copy to clipboard"
                >
                  <FaClipboard color="#fff" />
                </button>
                <SyntaxHighlighter
                  style={dracula}
                  PreTag="div"
                  language={match[1]}
                  {...props}
                >
                  {codeString}
                </SyntaxHighlighter>
              </div>
            ) : (
              <code className={className} {...props}>
                {children}
              </code>
            );
          },
        }}
      >
        {post.content}
      </Markdown>
      <button
        onClick={() => router.back()}
        className="text-purple-800 mt-4 inline-block hover:underline"
      >
        Back to Blogs
      </button>
    </div>
  );
};

export default BlogPostPage;


```
### Key Points

- In the `page.tsx` file, we created a dynamic blog post page using the `[slug]` dynamic route. This component allows us to fetch and display blog post content based on the slug from the URL. Here’s a breakdown of the main features:

- We use the `useEffect` hook to fetch the blog post data from an API using the `getPostBySlug` function, which takes the `slug` as a parameter. The fetched data is then stored in the component’s state.

- The blog content is written in markdown, and we use the `react-markdown` package to render it, supporting various markdown syntax features like **headings**, **lists**, and **links**. The `remark-gfm` plugin adds GitHub-flavored markdown, while `rehype-raw` allows rendering raw HTML.

- Code blocks within the blog are styled using `react-syntax-highlighter` with the `Dracula` theme. We also provide a button for users to copy code snippets to their clipboard.

- The component displays the blog’s `title`, `description`, `publication date` (formatted with `Moment.js`), `categories`, and `cover image` (if available).

- A *"Back to Blogs"* button allows users to navigate back to the blog listing.

This setup allows us to dynamically render blog posts, enhancing the user experience with features like *code copying* and *syntax highlighting*.


## Creating Write Post page

We will proceed to create a `WritePost` component to enable users seamlessly write posts and upload images.

Create a folder in the `app` directory called `write` and create a `page.tsx` file to create a new page with the route `/write`. 

Paste the following code in the `page.tsx`

```tsx
// app/write/page.tsx.
"use client";
import React, { useState } from "react";
import { useRouter } from "next/navigation";
import { FaArrowLeft } from "react-icons/fa";
import slugify from "react-slugify";
import MarkdownEditor from "@uiw/react-markdown-editor";
import Image from "next/image";
import { createPost, uploadImage } from "@/lib/api";
import { toast } from "react-hot-toast";

const WritePost = () => {
  const [markdownContent, setMarkdownContent] = useState("");
  const [title, setTitle] = useState("");
  const [description, setDescription] = useState("");
  const [coverImage, setCoverImage] = useState<File | null>(null);
  const [imagePreview, setImagePreview] = useState<string | null>(null);
  const [isLoading, setIsLoading] = useState(false); // Loading state
  const [error, setError] = useState<string | null>(null); // Error state
  const router = useRouter();

  // Handle image upload and preview
  const handleImageChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    if (e.target.files && e.target.files[0]) {
      const selectedImage = e.target.files[0];
      setCoverImage(selectedImage);
      setImagePreview(URL.createObjectURL(selectedImage));
    }
  };

  // Handle post submission
  const handleSubmit = async () => {
    setIsLoading(true);
    setError(null);
    try {
      // Create slug from the title
      const postSlug = slugify(title);
      // Create the post initially without the image
      const postData = {
        title,
        description,
        slug: postSlug,
        content: markdownContent,
      };

      // Step 1: Create the blog post without the cover image
      const postResponse = await createPost(postData);
      const postId = postResponse.id;
      console.log(postId);

      // Step 2: Upload cover image (if provided) and associate with blog post
      if (coverImage) {
        const uploadedImage = await uploadImage(coverImage, postId);
        console.log("Image uploaded:", uploadedImage);
      }

      // Redirect after successful post creation
      router.push(`/blogs/${postSlug}`);
      toast.success("Post created successfully");
    } catch (error) {
      console.error("Failed to create post:", error);
      setError("Failed to create post. Please try again.");
      toast.error("Failed to create post. Please try again.");
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="max-w-screen-md mx-auto p-4">
      <button
        onClick={() => router.back()}
        className="text-purple-400 hover:text-purple-500 mb-6 flex items-center space-x-2"
      >
        <FaArrowLeft /> <span>Back</span>
      </button>

      <h1 className="text-xl font-bold mb-4 text-gray-100 font-jet-brains">
        Create New Post
      </h1>
      {/* Render a message if there is an error */}
      {error && (
        <div className="mb-4 p-3 bg-red-600 text-white rounded-md">{error}</div>
      )}

      <div className="mb-4">
        <input
          type="text"
          placeholder="Enter a Title"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          className="w-full p-2 font-jet-brains text-3xl font-semibold bg-[#161b22] text-gray-100 border-b border-gray-600 focus:border-purple-500 focus:outline-none placeholder-gray-400"
        />
      </div>
      <div className="mb-4">
        <textarea
          placeholder="Description"
          value={description}
          onChange={(e) => setDescription(e.target.value)}
          className="w-full p-2 font-jet-brains bg-[#161b22] font-semibold text-gray-100 border-b border-gray-600 focus:border-purple-500 focus:outline-none placeholder-gray-400"
        />
      </div>
      <div className="mb-6">
        <input
          type="file"
          accept="image/*"
          onChange={handleImageChange}
          className="w-full bg-[#161b22] text-gray-100"
        />
        {imagePreview && (
          <div className="mt-4">
            <Image
              src={imagePreview}
              alt="Selected Cover"
              width="100"
              height="100"
              className="w-full h-auto rounded-md"
            />
          </div>
        )}
      </div>

      <div className="mb-6">
        <MarkdownEditor
          value={markdownContent}
          height="200px"
          onChange={(value) => setMarkdownContent(value)}
          className="bg-[#161b22] text-gray-100"
        />
      </div>

      <button
        onClick={handleSubmit}
        disabled={isLoading || (!title && !description)}
        className="bg-purple-600 text-gray-100 py-2 px-4 rounded-md hover:bg-purple-500"
      >
        {isLoading ? "Loading" : "Post"}
      </button>
    </div>
  );
};

export default WritePost;

```

The `WritePost` component allows users to write and submit a blog post, including uploading a cover image. Let's break down its key functionalities below:

### **1. State Management**
The component uses React’s `useState` hook to manage the following states:

- **`markdownContent`**: Stores the content of the blog post written in Markdown format.
- **`title`** and **`description`**: Manage the post's title and description, which will be displayed on the blog.
- **`coverImage`**: Holds the image file selected by the user for the post’s cover.
- **`imagePreview`**: Provides a URL to preview the selected cover image before submission.
- **`isLoading`**: Tracks whether the post submission is in progress. While loading, the submit button is disabled.
- **`error`**: Captures and displays any errors encountered during form submission (e.g., failed API requests).

### **2. Image Handling**
The component includes the **`handleImageChange`** function, which manages image uploads:

- When an image is selected, it saves the file to the `coverImage` state.
- A preview URL is generated using `URL.createObjectURL()`, allowing the user to see the image before submitting it.

This feature provides visual feedback, letting users know exactly what image they are uploading as the post’s cover.

### **3. Markdown Editor**
The post content is written in Markdown format using the **`@uiw/react-markdown-editor`** package. This editor simplifies text formatting for headings, lists, links, and more.

- The **`onChange`** event handler updates the `markdownContent` state as the user types, ensuring that all content is captured.

### **4. Post Submission**
The **`handleSubmit`** function processes the form submission in two steps:

1. **Create the Post**:
   - A **slug** is generated from the post title using `slugify(title)`. The slug is a URL-friendly version of the title (e.g., "My First Post" becomes "my-first-post").
   - The post data (title, description, slug, and markdown content) is sent to the server via the `createPost` function. The server responds with the post’s ID.

2. **Upload the Cover Image**:
   - If a cover image was selected, it is uploaded separately using the `uploadImage` function.
   - The image is linked to the post using its ID.

After successful creation and image upload:
- The user is redirected to the blog post’s page (`/blogs/${postSlug}`).
- A success message is displayed using **`toast.success()`**.
- If an error occurs, an error message is displayed using **`toast.error()`**.

### **5. User Experience**
- **Image Preview**: Users can preview the selected cover image before submitting their post.
- **Loading Indicator**: The `isLoading` state ensures that the submit button is disabled during submission to prevent duplicate posts.
- **Error Handling**: Any issues encountered during submission are displayed to the user through error messages.
- **Success Notification**: Upon successful submission, users receive a notification and are redirected to the new blog post.

### **Summary**
The `WritePost` component provides a comprehensive experience for users to write blog posts with Markdown formatting, upload a cover image, and submit the post to the server. It offers visual feedback (image previews), handles loading states, and ensures that errors are displayed when necessary. The blog post is created and saved in two steps: **content creation** and **image uploading**, ensuring that each part is handled correctly.

# Hosting Your Application With Strapi
Check the official documentation to read up on hosting your strapi application in more detail: [Strapi Hosting Documentation](https://docs.strapi.io/dev-docs/deployment).

# Conclusion

In this guide, we explored how to build a developer-friendly blog application using Next.js and Strapi. From setting up our development environment to creating content models, setting up the application layout, creating dynamic routes for individual blog posts, we covered how to effectively manage state, implement pagination and search functionality, handle image uploads with strapi, and implement a markdown editor for content creation. 

Feel free to clone the application from the [GitHub repository](https://github.com/chrismbah/strapi-dev-blog) and extend its functionality. To learn more about Strapi and its other impressive features, you can visit the [official documentation](https://docs.strapi.io/).

