# 🎓 Complete Beginner's Guide to Agility Content Fetch JS SDK

**Author:** GitHub Copilot (expanded by Claude)
**Created:** 2026-03-04 | **Updated:** 2026-03-04
**Difficulty Level:** Beginner
**Estimated Learning Time:** 10-14 days (1-2 hours per day)

---

## Table of Contents

1. [What is this SDK?](#part-1-what-is-this-sdk-and-why-does-it-exist)
2. [Repository Structure](#part-2-repository-structure-breakdown)
3. [How Everything Works Together](#part-3-how-everything-works-together)
4. [Core Concepts](#part-4-core-concepts)
5. [Available Methods](#part-5-available-methods-what-you-can-do)
6. [Learning Plan](#part-6-learning-plan-step-by-step)
7. [Key Files to Study](#part-7-key-files-to-study-in-priority-order)
8. [Common Patterns](#part-8-common-patterns-youll-see)
9. [Questions to Ask Yourself](#part-9-questions-to-ask-yourself-while-learning)
10. [Quick Reference](#part-10-quick-reference---the-most-important-files)
11. [Installation Methods](#part-11-installation-methods) ⭐ NEW
12. [Contributing to the SDK](#part-12-contributing-to-the-sdk) ⭐ NEW
13. [Content Webhooks](#part-13-content-webhooks) ⭐ NEW
14. [Official Tutorials & Resources](#part-14-official-tutorials--resources) ⭐ NEW
15. [SDK Versioning & Releases](#part-15-sdk-versioning--releases) ⭐ NEW

---

## Part 1: What is this SDK and Why Does It Exist?

The **Agility Content Fetch JS SDK** is a JavaScript/TypeScript library that acts as a **bridge between your web application and Agility CMS**. Think of it as a translator:

```
Your Web App → (Fetch SDK) → Agility CMS Server → Gets Content Data → Returns to App
```

### Real-world Analogy

If Agility CMS is a restaurant kitchen storing recipes and ingredients, this SDK is like a waiter who knows:

* Exactly which kitchen door to knock on
* What to ask for
* How to bring the food back to your table in the right format

### What Problem Does It Solve?

**Without SDK:** You'd have to:

```javascript
// Manual approach - complicated!
const response = await fetch('https://api.agilitycms.com/fetch/en-us/item/22', {
    headers: {
        'APIKey': 'your-secret-key',
        'Guid': 'your-guid'
    }
})
const data = await response.json()
// Now you have to validate and type the data yourself...
```

**With SDK:** Much simpler!

```javascript
const api = agility.getApi({ guid: '...', apiKey: '...' })
const item = await api.getContentItem({ contentID: 22, locale: 'en-us' })
// Data is already typed and validated!
```

### What This SDK Is NOT

Understanding the boundaries of this SDK is just as important as knowing what it does:

- **It is NOT a write/management SDK.** It can only READ content. If you need to create, update, or delete content programmatically, you'd use the separate `@agility/content-management` or `@agility/management-sdk` packages.
- **It is NOT a sync SDK.** For caching content locally (e.g. to a database or filesystem), there is a dedicated `@agility/content-sync` package that builds on top of this one.
- **It is NOT a CMS UI.** The CMS interface itself lives at `app.agilitycms.com` — this SDK only handles data retrieval.

### Node.js vs. Browser

One important feature of this SDK is that it works in **both environments**:

- **Node.js** — server-side rendering (Next.js, Express, etc.)
- **Browser** — client-side SPAs (React, Vue, plain HTML)

This is intentional. The SDK handles environment detection internally, so the same code works in both places without changes.

---

## Part 2: Repository Structure Breakdown

Here's how the repo is organized:

```
agility-content-fetch-js-sdk/
├── src/                          # Source code (what you care about)
│   ├── index.ts                  # Main entry point (1 file, ~30 lines)
│   ├── api-client.ts             # The heart - handles all requests (~300 lines)
│   ├── utils.ts                  # Helper functions (~120 lines)
│   ├── methods/                  # Individual API methods (10 files)
│   │   ├── getPage.ts
│   │   ├── getContentItem.ts
│   │   ├── getContentList.ts
│   │   ├── getSitemapFlat.ts
│   │   ├── getSitemapNested.ts
│   │   ├── getGallery.ts
│   │   ├── getSyncContent.ts
│   │   ├── getSyncPages.ts
│   │   ├── getPageByPath.ts
│   │   └── getUrlRedirections.ts
│   └── types/                    # TypeScript definitions (15+ files)
│       ├── Page.ts
│       ├── ContentItem.ts
│       ├── Config.ts
│       ├── Client.ts
│       ├── ContentZone.ts
│       ├── FilterOperator.ts
│       └── ... (other types)
├── test/                         # Unit tests
├── .github/workflows/            # CI/CD pipeline definitions ⭐ NEW
├── package.json                  # Dependencies & scripts
├── README.md                     # Official documentation
├── tsconfig.json                 # TypeScript configuration
├── jest.config.js                # Test configuration
├── build.js                      # Build script
└── yarn.lock                     # Yarn dependency lock file ⭐ NEW
```

### Folder-by-Folder Breakdown

#### **1. `src/index.ts` - The Front Door**

This is the ONLY file exported to users. Everything flows through here.

```typescript
import { Config } from "./types/Config"
import { ContentItem } from "./types/ContentItem"
// ... other imports ...

import { ApiClientInstance, getApi } from "./api-client";

export type {
    ApiClientInstance,
    Config,
    ContentItem,
    // ... other types ...
}

export {
    getApi
}

module.exports = {
    getApi
}
```

**Why?** Clean API - users only know about one function: `getApi()`

---

#### **2. `src/api-client.ts` - The Brain**

This is the most important file. Size: ~300 lines. Contains:

**A) Default Configuration:**

```typescript
const defaultConfig: Config = {
    baseUrl: null,
    isPreview: false,
    guid: null,
    apiKey: null,
    locale: null,
    headers: {},
    requiresGuidInHeaders: false,
    logLevel: 'warn',
    debug: false,
    caching: {
        maxAge: 0  // caching disabled by default
    }
};
```

**B) The ApiClient Class:**

```typescript
class ApiClient {
    config: Config;
    
    constructor(config: Config) {
        // Validate and store config
        this.config = config;
    }
    
    // All these methods:
    async getContentItem(params) { ... }
    async getContentList(params) { ... }
    async getPage(params) { ... }
    async getPageByPath(params) { ... }
    async getSitemapFlat(params) { ... }
    async getSitemapNested(params) { ... }
    async getGallery(params) { ... }
    async getSyncContent(params) { ... }
    async getSyncPages(params) { ... }
    async getUrlRedirections(params) { ... }
    
    // The workhorse:
    async makeRequest(reqConfig) {
        // Uses fetch() API to make HTTP request
        const response = await fetch(fullUrl, init)
        const data = await response.json()
        return data
    }
}
```

**C) The getApi() Function:**

```typescript
export function getApi(config: Config) {
    validateConfigParams(config);
    if (!config.fetchConfig) config.fetchConfig = {}
    return new ApiClient(config);
}
```

**How it works:**

1. Takes your config (guid, apiKey, etc.)
2. Validates it
3. Creates and returns a new ApiClient instance
4. You now have access to all the methods!

---

#### **3. `src/methods/` - The Toolbox**

10 method files. Each does ONE thing. Each follows the same pattern.

**Example: `getContentItem.ts`**

```typescript
// 1. Define what parameters are needed
export interface ContentItemRequestParams {
    contentID: number;          // REQUIRED
    locale?: string;            // OPTIONAL
    languageCode?: string;      // OPTIONAL (deprecated)
    contentLinkDepth?: number;  // OPTIONAL
    expandAllContentLinks?: boolean;  // OPTIONAL
}

// 2. Default values for optional params
const defaultParams = {
    contentLinkDepth: 1,
    expandAllContentLinks: false
}

// 3. The function
function getContentItem<T>(
    this: ApiClientInstance,
    requestParams: ContentItemRequestParams
): Promise<ContentItem<T>> {
    
    // STEP 1: Validate
    validateRequestParams(requestParams);
    
    // STEP 2: Apply defaults
    requestParams = { ...defaultParams, ...requestParams };
    
    // STEP 3: Build the request object
    const req = {
        url: `/item/${requestParams.contentID}?contentLinkDepth=${requestParams.contentLinkDepth}&expandAllContentLinks=${requestParams.expandAllContentLinks}`,
        method: 'get',
        baseURL: buildRequestUrlPath(this.config, requestParams.locale),
        headers: buildAuthHeader(this.config),
        params: {}
    };
    
    // STEP 4: Make the actual request
    return this.makeRequest(req);
}

// 4. Validation function
function validateRequestParams(requestParams: ContentItemRequestParams) {
    if (!requestParams.locale && !requestParams.languageCode) {
        throw new TypeError('You must include a locale in your request params.')
    }
    if (!requestParams.contentID) {
        throw new TypeError('You must include a contentID number in your request params.');
    }
}

export default getContentItem;
```

**This pattern repeats for every method** - just different URLs and validations.

---

#### **4. `src/types/` - The Blueprints**

Defines the shape of data. These are TypeScript interfaces that describe what you get back.

**Example: `ContentItem.ts`**

```typescript
interface ContentItemProperties {
  state: number                  // Status of item
  modified: Date                 // When it was last changed
  versionID: number              // Version number
  referenceName: string          // Type/category
  definitionName: string         // How it's defined
  itemOrder: number              // Order in list
}

export interface ContentItem<T = { [key: string]: any }> {
  contentID: number;             // The unique ID
  properties: ContentItemProperties;  // Metadata
  fields: T;                     // YOUR ACTUAL CONTENT
  seo?: SEOProperties;           // Optional SEO info
}
```

**Example: `Page.ts`**

```typescript
export interface Page {
  pageID: number;
  name: string;
  path: string;
  title: string;
  menuText: string;
  pageType: "static" | "dynamic" | "dynamic_node" | "link" | "folder";
  templateName: string;
  securePage: boolean;
  properties: SystemProperties;
  zones: { [key: string]: ContentZone[] };  // ZONES on the page
  redirectUrl?: string;
  dynamicItemContentID?: number;
  visible: SitemapVisibility;
  seo?: SEOProperties;
}
```

**Key types include:**

* `ContentItem.ts` - A piece of content
* `Page.ts` - A web page
* `ContentZone.ts` - Area on a page where content goes
* `Gallery.ts` - Collection of images
* `Config.ts` - Configuration options
* `FilterOperator.ts` - How to filter results

---

#### **5. `src/utils.ts` - Helper Functions**

Utility functions used throughout the SDK.

```typescript
// Builds authentication headers for API calls
function buildAuthHeader(config) {
    let defaultAuthHeaders = {
        APIKey: config.apiKey,
        Guid: ''
    };
    
    if (config.requiresGuidInHeaders) {
        defaultAuthHeaders.Guid = config.guid;
    }
    
    return { ...defaultAuthHeaders, ...config.headers }
}

// Builds the base URL path for requests
function buildRequestUrlPath(config, locale) {
    let apiFetchOrPreview = config.isPreview ? 'preview' : 'fetch';
    let urlPath = `${config.baseUrl}/${apiFetchOrPreview}/${locale}`;
    return urlPath;
}

// Logging functions
function logDebug({ config, message }: LogProps) {
    if (config.logLevel !== 'debug' && !config.debug) return
    console.log('\x1b[33m%s\x1b[0m', message);
}

function logError({ config, message }: LogProps) {
    if (config.logLevel === 'silent') return
    console.error('\x1b[41m%s\x1b[0m', message);
}
```

---

#### **6. `.github/workflows/` - CI/CD Pipelines** ⭐ NEW

This folder contains GitHub Actions workflow files that automatically run tests whenever code is pushed or a pull request is opened. You'll often see build status badges in the README like this:

```
[![Build Status](https://agility.visualstudio.com/.../badge)](...)
[![Netlify Status](https://api.netlify.com/.../badge)](...)
```

These badges show in real time whether the tests are passing and whether the docs site is deployed. As a beginner you don't need to modify these, but knowing they exist explains why the project is reliable — every change is automatically tested before it merges.

---

## Part 3: How Everything Works Together

### The Complete Flow (Step by Step)

```
STEP 1: User imports
┌─────────────────────────────────────┐
│ import agility from                 │
│   '@agility/content-fetch'          │
│ src/index.ts exports getApi         │
└────────────────┬────────────────────┘
                 │
                 ▼
STEP 2: Create client
┌─────────────────────────────────────┐
│ const api = agility.getApi({        │
│   guid: 'xxx',                      │
│   apiKey: 'yyy'                     │
│ })                                  │
│                                     │
│ Calls: getApi() in api-client.ts    │
│ Returns: new ApiClient(config)      │
└────────────────┬────────────────────┘
                 │
                 ▼
STEP 3: Call a method
┌─────────────────────────────────────┐
│ api.getContentItem({                │
│   contentID: 22,                    │
│   locale: 'en-us'                   │
│ })                                  │
│                                     │
│ Routes to: methods/getContentItem.ts│
└────────────────┬────────────────────┘
                 │
                 ▼
STEP 4: Method execution
┌─────────────────────────────────────┐
│ 1. Validate parameters              │
│    - Check contentID exists         │
│    - Check locale exists            │
│                                     │
│ 2. Apply defaults                   │
│    contentLinkDepth: 1              │
│    expandAllContentLinks: false     │
│                                     │
│ 3. Build request object             │
│    url: /item/22?...                │
│    headers: { APIKey, Guid }        │
│    baseURL: https://api...          │
│                                     │
│ 4. Call makeRequest(req)            │
└────────────────┬────────────────────┘
                 │
                 ▼
STEP 5: HTTP Request
┌─────────────────────────────────────┐
│ makeRequest() in api-client.ts:     │
│                                     │
│ fetch(                              │
│   'https://api.agilitycms.com'      │
│     + '/fetch'                      │
│     + '/en-us'                      │
│     + '/item/22?...'                │
│   {                                 │
│     method: 'GET',                  │
│     headers: { APIKey, Guid, ... }  │
│   }                                 │
│ )                                   │
└────────────────┬────────────────────┘
                 │
                 ▼
STEP 6: Server Response
┌─────────────────────────────────────┐
│ Agility CMS API Server:             │
│                                     │
│ Returns JSON:                       │
│ {                                   │
│   "contentID": 22,                  │
│   "properties": { ... },            │
│   "fields": { ... },                │
│   "seo": { ... }                    │
│ }                                   │
└────────────────┬────────────────────┘
                 │
                 ▼
STEP 7: Return typed data
┌─────────────────────────────────────┐
│ Parse response                      │
│ Cast to ContentItem<T>              │
│ Return as Promise                   │
│                                     │
│ User code receives:                 │
│ {                                   │
│   contentID: 22,                    │
│   properties: { ... },              │
│   fields: { ... },                  │
│   seo: { ... }                      │
│ }                                   │
└─────────────────────────────────────┘
```

### Simplified Flow Diagram

```
YOUR APP
    │
    ├─→ import agility
    │
    ├─→ getApi({ guid, apiKey })  ──→ [src/index.ts] ──→ [src/api-client.ts]
    │                                   exports getApi()
    │
    ├─→ api.getContentItem({...})  ──→ [src/api-client.ts]
    │                                   │
    │                                   ├─→ [src/methods/getContentItem.ts]
    │                                   │   Validate → Build → Call makeRequest()
    │                                   │
    │                                   ├─→ [src/utils.ts]
    │                                   │   buildAuthHeader()
    │                                   │   buildRequestUrlPath()
    │                                   │
    │                                   └─→ fetch() API
    │                                       HTTP GET Request
    │
    └─→ await response  ←─ [types/ContentItem.ts] ←─ JSON from server
        Typed data!
```

---

## Part 4: Core Concepts

### 1. Configuration (Config)

The config object tells the SDK how to behave.

```typescript
// src/types/Config.ts

export interface Config {
  // **REQUIRED**
  guid?: string | null;           // Your instance ID
  apiKey?: string | null;         // Your secret key

  // **SEMI-REQUIRED** (Defaults provided)
  baseUrl?: string | null;        // API endpoint (auto-determined)
  isPreview?: boolean;            // Use preview or fetch API

  // **OPTIONAL**
  locale?: string | null;         // Default language
  headers?: { [key: string]: string };  // Extra HTTP headers
  requiresGuidInHeaders?: boolean; // Include Guid in headers?
  logLevel?: 'debug' | 'info' | 'warn' | 'error' | 'silent';
  debug?: boolean;                // Show response headers?
  caching?: {
    maxAge?: number;              // Cache duration in ms
  };
  fetchConfig?: any;              // Next.js caching options
}
```

**Example:**

```typescript
const api = agility.getApi({
    guid: 'abc123',
    apiKey: 'secret-key-here',
    isPreview: false,           // Use live content
    logLevel: 'warn',           // Only log warnings/errors
    caching: {
        maxAge: 180000          // Cache for 3 minutes
    }
})
```

### 2. Live vs. Preview API

One of the most important `Config` options is `isPreview`. This toggles which Agility endpoint the SDK calls:

| `isPreview` | Endpoint used | Content returned |
|---|---|---|
| `false` (default) | `/fetch/...` | Published, live content only |
| `true` | `/preview/...` | Draft + unpublished content |

**When to use preview mode:**
- During content editing workflows so editors can see unpublished changes
- In a staging/preview deployment

```typescript
// Live site
const liveApi = agility.getApi({ guid, apiKey, isPreview: false })

// Preview site (for editors)
const previewApi = agility.getApi({ guid, apiKey: previewApiKey, isPreview: true })
```

> ⚠️ **Security note:** Use a different `apiKey` for preview vs. live. Agility CMS provides separate API keys for each. Never expose your preview API key publicly.

### 3. Request Parameters

Each method takes specific parameters. Structure:

```typescript
{
    requiredParam: value,           // Must provide
    optionalParam?: value,          // Provide or use default
    anotherOptional?: value
}
```

**Example from `getContentItem`:**

```typescript
await api.getContentItem({
    contentID: 22,                  // REQUIRED
    locale: 'en-us',               // REQUIRED
    contentLinkDepth: 1,            // OPTIONAL (default: 1)
    expandAllContentLinks: false    // OPTIONAL (default: false)
})
```

**What these mean:**

* `contentID`: Which piece of content to get
* `locale`: Which language/region (en-us, fr-ca, etc)
* `contentLinkDepth`: How many levels deep to fetch linked content
* `expandAllContentLinks`: Expand all references, even complex ones?

### 4. Understanding `contentLinkDepth`

This parameter deserves special attention because it directly affects performance and the shape of the data you receive.

When Agility content items reference each other (e.g. a Blog Post references an Author content item), the SDK can automatically resolve those links for you.

```typescript
// contentLinkDepth: 0 → only the item itself, no linked items resolved
// contentLinkDepth: 1 → resolves direct links (default, usually sufficient)
// contentLinkDepth: 2 → resolves links-of-links
// contentLinkDepth: 3+ → deep nesting, use sparingly — increases response time
```

**Example:**

```typescript
// Blog post links to an Author item
const post = await api.getContentItem({
    contentID: 22,
    locale: 'en-us',
    contentLinkDepth: 1  // default: Author object is resolved inline
})

// post.fields.author is now a full ContentItem object, not just an ID
console.log(post.fields.author.fields.name)  // "Jane Doe"
```

### 5. Response Types

Data comes back as TypeScript interfaces. You get autocomplete and type safety.

```typescript
// What you request:
api.getContentItem({ contentID: 22, locale: 'en-us' })

// What you get back:
Promise<ContentItem<T>>

// Which looks like:
{
    contentID: 22,
    properties: {
        state: 1,
        modified: 2026-03-04T10:00:00.000Z,
        versionID: 5,
        referenceName: 'posts',
        definitionName: 'Blog Post',
        itemOrder: 1
    },
    fields: {                           // YOUR CUSTOM CONTENT
        title: "How to Learn APIs",
        body: "Lorem ipsum...",
        author: "John Doe",
        publishDate: "2026-03-04"
    },
    seo: {
        metaDescription: "Learn APIs easily",
        metaKeywords: "api, rest, learning",
        metaHTML: "",
        menuVisible: true,
        sitemapVisible: true
    }
}
```

### 6. Error Handling

Errors happen at two stages:

**Stage 1: Parameter Validation** (before API call)

```typescript
// This throws immediately
api.getContentItem({
    // Missing contentID!
    locale: 'en-us'
})

// Error: "You must include a contentID number in your request params."
```

**Stage 2: API Response** (after API call)

```typescript
// API returns 404 - item doesn't exist
// SDK logs info message and returns: undefined

// API returns 500 - server error
// SDK logs error message and returns: undefined

// Network error
// SDK logs error and throws exception
```

**Best practice — always wrap API calls in try/catch:**

```typescript
try {
    const item = await api.getContentItem({ contentID: 22, locale: 'en-us' })
    if (!item) {
        console.warn('Item not found')
        return null
    }
    return item
} catch (error) {
    console.error('Network or unexpected error:', error)
    return null
}
```

### 7. Caching

Optional. Stores responses in memory.

```typescript
const api = agility.getApi({
    guid: 'abc123',
    apiKey: 'key',
    caching: {
        maxAge: 180000  // 3 minutes
    }
})

// First call: Makes API request, stores result
await api.getContentItem({ contentID: 22, locale: 'en-us' })

// Second call (within 3 mins): Returns cached result
await api.getContentItem({ contentID: 22, locale: 'en-us' })

// Third call (after 3 mins): Makes new API request
await api.getContentItem({ contentID: 22, locale: 'en-us' })
```

**When should you use caching?**

| Scenario | Recommended `maxAge` |
|---|---|
| Development / debugging | `0` (disabled) |
| Static site generation | `0` (always fresh at build time) |
| Server-side rendering (SSR) | `60000–180000` (1–3 min) |
| Client-side SPA | `120000–300000` (2–5 min) |

> Note: Caching is **in-memory only** — it resets whenever your server or browser tab restarts. For persistent caching across server restarts, consider the `@agility/content-sync` package.

---

## Part 5: Available Methods (What You Can Do)

### Overview Table

| Method | Purpose | Returns | When to Use |
| --- | --- | --- | --- |
| **getPage()** | Get a single page by ID | `Promise<Page>` | You know the page ID |
| **getPageByPath()** | Get a page by URL path | `Promise<Page>` | You have the page path |
| **getContentItem()** | Get a single content item | `Promise<ContentItem>` | You know the content ID |
| **getContentList()** | Get multiple items with filters | `Promise<ContentList>` | You want filtered/paginated items |
| **getSitemapFlat()** | Get all pages as flat list | `Promise<SitemapFlat>` | For site routing |
| **getSitemapNested()** | Get pages as tree structure | `Promise<SitemapNested>` | For menu generation |
| **getGallery()** | Get media items | `Promise<Gallery>` | For images/media |
| **getSyncContent()** | Get changed content | `Promise<SyncContent>` | Keep app in sync |
| **getSyncPages()** | Get changed pages | `Promise<SyncPages>` | Keep pages in sync |
| **getUrlRedirections()** | Get redirect rules | `Promise<UrlRedirections>` | Handle old URLs |

### Detailed Examples

#### **1. getPage() - Get a Single Page**

```typescript
const page = await api.getPage({
    pageID: 1,                          // REQUIRED
    locale: 'en-us',                    // REQUIRED
    contentLinkDepth: 2,                // OPTIONAL
    expandAllContentLinks: true         // OPTIONAL
})

// Returns:
{
    pageID: 1,
    name: 'home',
    path: '/',
    title: 'Home Page',
    menuText: 'Home',
    pageType: 'static',
    templateName: 'Homepage Template',
    securePage: false,
    properties: { ... },
    zones: {
        'hero': [{
            module: 'hero',
            item: { contentID: 5, fields: { ... } }
        }],
        'content': [{
            module: 'text',
            item: { contentID: 6, fields: { ... } }
        }]
    }
}
```

**Use when:** You know the page ID (like in a dynamic route)

---

#### **2. getPageByPath() - Get a Page by URL**

```typescript
const page = await api.getPageByPath({
    pagePath: '/about',                 // REQUIRED
    channelName: 'website',             // REQUIRED
    locale: 'en-us',                    // REQUIRED
    contentLinkDepth: 2,                // OPTIONAL
    expandAllContentLinks: true         // OPTIONAL
})

// Returns: Same as getPage()
```

**Use when:** You have the URL path (like `/blog/my-post`)

---

#### **3. getContentItem() - Get a Single Content Item**

```typescript
const item = await api.getContentItem({
    contentID: 22,                      // REQUIRED
    locale: 'en-us',                    // REQUIRED
    contentLinkDepth: 1,                // OPTIONAL (default)
    expandAllContentLinks: false        // OPTIONAL (default)
})

// Returns:
{
    contentID: 22,
    properties: {
        state: 1,
        modified: 2026-03-04T10:00:00Z,
        versionID: 5,
        referenceName: 'blog_posts',
        definitionName: 'Blog Post'
    },
    fields: {
        title: "My Blog Post",
        body: "Post content here...",
        author: "John Doe"
    }
}
```

**Use when:** You want a single piece of content

---

#### **4. getContentList() - Get Multiple Items**

```typescript
const items = await api.getContentList({
    referenceName: 'blog_posts',        // REQUIRED - content type
    locale: 'en-us',                    // REQUIRED
    
    // Optional pagination
    take: 10,                           // How many items (default: 10)
    skip: 0,                            // How many to skip (for pagination)
    
    // Optional sorting
    sort: 'properties.modified',        // Field to sort by
    direction: 'DESC',                  // ASC or DESC
    
    // Optional filtering
    filters: [
        {
            property: 'fields.status',
            operator: 'eq',
            value: '"published"'
        },
        {
            property: 'fields.author',
            operator: 'eq',
            value: '"John Doe"'
        }
    ],
    filtersLogicOperator: 'AND'         // AND or OR
})

// Returns:
{
    pageNumber: 1,
    pageSize: 10,
    totalCount: 50,
    items: [
        { contentID: 22, properties: {...}, fields: {...} },
        { contentID: 23, properties: {...}, fields: {...} },
        // ... 10 items total
    ]
}
```

**Use when:** You want filtered, paginated lists

**Filter operators:**

* `eq` - equals
* `ne` - not equals
* `lt` - less than
* `lte` - less than or equal
* `gt` - greater than
* `gte` - greater than or equal
* `like` - contains (string)
* `in` - matches any in list
* `contains` - contains (for arrays)
* `range` - between two values

**Pagination example:**

```typescript
// Page 1 (items 1-10)
const page1 = await api.getContentList({ referenceName: 'posts', locale: 'en-us', take: 10, skip: 0 })

// Page 2 (items 11-20)
const page2 = await api.getContentList({ referenceName: 'posts', locale: 'en-us', take: 10, skip: 10 })

// Total pages
const totalPages = Math.ceil(page1.totalCount / 10)
```

---

#### **5. getSitemapFlat() - Get All Pages as Flat List**

```typescript
const sitemap = await api.getSitemapFlat({
    channelName: 'website',             // REQUIRED
    locale: 'en-us'                     // REQUIRED
})

// Returns:
{
    '/': { pageID: 1, name: 'home', path: '/', ... },
    '/about': { pageID: 2, name: 'about', path: '/about', ... },
    '/contact': { pageID: 3, name: 'contact', path: '/contact', ... },
    '/blog': { pageID: 4, name: 'blog', path: '/blog', ... }
}
```

**Use when:** You need page routing/navigation or want to generate static paths in Next.js:

```typescript
// In Next.js getStaticPaths:
const sitemap = await api.getSitemapFlat({ channelName: 'website', locale: 'en-us' })
const paths = Object.keys(sitemap).map(path => ({ params: { slug: path.split('/').filter(Boolean) } }))
```

---

#### **6. getSitemapNested() - Get Pages as Tree**

```typescript
const sitemap = await api.getSitemapNested({
    channelName: 'website',             // REQUIRED
    locale: 'en-us'                     // REQUIRED
})

// Returns:
[
    {
        pageID: 1,
        path: '/',
        title: 'Home',
        children: []
    },
    {
        pageID: 4,
        path: '/blog',
        title: 'Blog',
        children: [
            {
                pageID: 10,
                path: '/blog/post-1',
                title: 'My First Post',
                children: []
            }
        ]
    }
]
```

**Use when:** You need hierarchical menus

---

#### **7. getGallery() - Get Media Items**

```typescript
const gallery = await api.getGallery({
    galleryID: 5                        // REQUIRED
})

// Returns:
{
    galleryID: 5,
    name: 'Homepage Images',
    description: 'Images for homepage',
    count: 10,
    media: [
        {
            mediaID: 1,
            label: 'Hero Image',
            url: 'https://...',
            metadata: {
                Title: 'Hero Image',
                Description: 'Main hero',
                LinkUrl: '',
                pixelHeight: '1080',
                pixelWidth: '1920'
            }
        }
    ]
}
```

**Use when:** You want to display images/media

---

#### **8. getSyncContent() - Keep Content in Sync**

```typescript
// Start a new sync
let syncResponse = await api.getSyncContent({
    syncToken: '0',                     // Start from beginning
    locale: 'en-us',
    pageSize: 500
})

// Get changed items
console.log(syncResponse.items)       // Array of changed items
console.log(syncResponse.syncToken)   // Token for next call

// Continue syncing until caught up
while (syncResponse.syncToken) {
    syncResponse = await api.getSyncContent({
        syncToken: syncResponse.syncToken,
        locale: 'en-us'
    })
}
```

**Use when:** You need to sync changes to a database

---

#### **9. getSyncPages() - Keep Pages in Sync**

Similar to getSyncContent(), but for pages.

```typescript
let syncResponse = await api.getSyncPages({
    syncToken: '0',
    locale: 'en-us'
})

while (syncResponse.syncToken) {
    syncResponse = await api.getSyncPages({
        syncToken: syncResponse.syncToken,
        locale: 'en-us'
    })
}
```

---

#### **10. getUrlRedirections() - Handle Old URLs**

```typescript
const redirects = await api.getUrlRedirections({
    lastAccessDate: new Date('2026-01-01')  // OPTIONAL
})

// Returns:
{
    urlRedirections: [
        { oldUrl: '/old-page', newUrl: '/new-page', statusCode: 301 },
        { oldUrl: '/blog/post-1', newUrl: '/articles/post-1', statusCode: 301 }
    ],
    lastAccessDate: 2026-03-04T10:00:00Z
}
```

**Use when:** You need to handle redirects (e.g. in a Next.js middleware or Express route)

---

## Part 6: Learning Plan (Step by Step)

### Phase 1: Foundation (Days 1-2)

**Goal:** Understand the basics
**Time:** 2-3 hours total

**Tasks:**

* Read the README.md in the repo
* Understand what Agility CMS is (headless content management)
* Understand what "fetch API" means (retrieve/read content)
* Set up the SDK in a test project (see Part 11 for install options)
* Create your first API client:

```typescript
import agility from '@agility/content-fetch'
const api = agility.getApi({
    guid: 'your-guid',
    apiKey: 'your-api-key'
})
```

**Resources:**

* Official README: https://github.com/agility/agility-content-fetch-js-sdk
* Agility docs: https://help.agilitycms.com/hc/en-us
* About Headless CMS: https://www.agilitycms.com/resources/agility-headless-cms

**Self-Check:**

* Can you explain what the SDK does in one sentence?
* Can you set up the SDK with your credentials?

---

### Phase 2: Core Files (Days 3-4)

**Goal:** Understand code structure
**Time:** 2-3 hours

**Read these files in order:**

1. **`src/index.ts`** (5 min) — Why is this file so small? Answer: Hides complexity, exports one function
2. **`src/types/Config.ts`** (10 min) — What configuration options are required? Answer: guid and apiKey
3. **`src/api-client.ts`** (20 min) — Focus on lines 1–76 first (getApi function), then lines 135–160 (method definitions). Don't worry about makeRequest yet.
4. **`src/utils.ts`** (15 min) — What does buildAuthHeader do? What does buildRequestUrlPath do?

**Self-Check:**

* Can you explain what getApi() does?
* Can you explain what makeRequest() is?
* Can you explain what buildAuthHeader() does?

---

### Phase 3: Methods (Days 5-7)

**Goal:** Understand how to fetch data
**Time:** 3-4 hours

**Pick these methods in order:**

1. `methods/getContentItem.ts` — Simplest method, focus on validate → build → request
2. `methods/getPage.ts` — Similar to getContentItem, different endpoint
3. `methods/getContentList.ts` — More complex, has filtering: new concepts include skip, take, sort, filters
4. `methods/getSitemapFlat.ts` — Different use case, simpler parameters

**For each method ask yourself:** What parameters does it take? What does it validate? What URL does it build? What does it return?

---

### Phase 4: Type System (Days 8-9)

**Goal:** Understand data structures
**Time:** 2-3 hours

**Study these type files in order:** `Config.ts` → `ContentItem.ts` → `Page.ts` → `ContentZone.ts` → `FilterOperator.ts`

**Exercise:** Create a TypeScript interface for your own content type and use it:

```typescript
interface BlogPost {
    title: string;
    body: string;
    author: string;
    publishDate: Date;
    tags: string[];
}

const post = await api.getContentItem<BlogPost>({
    contentID: 22,
    locale: 'en-us'
})

console.log(post.fields.title)  // TypeScript knows it's a string!
console.log(post.fields.tags)   // TypeScript knows it's string[]!
```

---

### Phase 5: Hands-On Practice (Days 10+)

**Goal:** Use the SDK in a real project
**Time:** 5-10 hours

**Milestones:**

* ✅ Create API client
* ✅ Fetch single content item
* ✅ Fetch list of items
* ✅ Filter the list (by status, author, etc)
* ✅ Get a page with its zones
* ✅ Display all data in UI
* ✅ Add loading states
* ✅ Add error handling
* ✅ Implement pagination (skip / take)
* ✅ Try preview mode (`isPreview: true`)

---

## Part 7: Key Files to Study (In Priority Order)

### Priority 1 - MUST READ (2-3 hours total)

| File | Time | Why Important | Key Concept |
| --- | --- | --- | --- |
| `README.md` | 10 min | Overview | What the SDK is |
| `src/index.ts` | 5 min | Entry point | Only exports getApi() |
| `src/api-client.ts` (lines 1-76) | 20 min | Initialization | How getApi() works |
| `src/api-client.ts` (lines 135-160) | 15 min | Methods | All available methods |
| `src/types/Config.ts` | 10 min | Configuration | What to pass to getApi() |
| `src/methods/getContentItem.ts` | 10 min | Simple example | How methods work |

### Priority 2 - IMPORTANT (2-3 hours total)

| File | Time | Why Important | Key Concept |
| --- | --- | --- | --- |
| `src/methods/getPage.ts` | 10 min | Page fetching | Similar pattern |
| `src/methods/getContentList.ts` | 15 min | Filtering | Most complex |
| `src/utils.ts` | 15 min | Helpers | URL & header building |
| `src/types/Page.ts` | 10 min | Page structure | Zones concept |
| `src/types/ContentItem.ts` | 10 min | Item structure | Generic types |
| `src/types/ContentZone.ts` | 5 min | Zone structure | Content placement |

### Priority 3 - REFERENCE (1-2 hours total)

| File | Time | When Needed |
| --- | --- | --- |
| `src/methods/getSitemapFlat.ts` | 5 min | Building routing |
| `src/methods/getSitemapNested.ts` | 5 min | Building menus |
| `src/methods/getGallery.ts` | 5 min | Displaying media |
| `src/methods/getSyncContent.ts` | 10 min | Syncing content |
| `src/types/FilterOperator.ts` | 5 min | Advanced filtering |
| `src/types/Client.ts` | 5 min | Type definitions |
| `test/` folder | 15 min | See how the SDK is tested |

---

## Part 8: Common Patterns You'll See

### Pattern 1: All Methods Follow This Structure

```typescript
// Every method in src/methods/ follows this:

// 1. Define parameters interface
export interface SomethingRequestParams {
    requiredParam: string;
    optionalParam?: number;
}

// 2. Define default values
const defaultParams = { optionalParam: 10 }

// 3. Implement the method
function getSomething(
    this: ApiClientInstance,
    requestParams: SomethingRequestParams
): Promise<Something> {
    validateRequestParams(requestParams);              // STEP A
    requestParams = { ...defaultParams, ...requestParams }; // STEP B
    const req = { url: `...`, method: 'get', ... };   // STEP C
    return this.makeRequest(req);                      // STEP D
}

// 4. Validation function
function validateRequestParams(requestParams) {
    if (!requestParams.requiredParam) {
        throw new TypeError('Required param missing');
    }
}

export default getSomething;
```

### Pattern 2: Parameters Build the URL

```
https://api.agilitycms.com/fetch/en-us/item/22?contentLinkDepth=1
                        │      │      │   │
                        │      │      │   └── contentID
                        │      │      └────── locale
                        │      └────────────── 'fetch' or 'preview'
                        └───────────────────── baseUrl
```

### Pattern 3: Promises for Async Operations

```typescript
// async/await (recommended)
try {
    const item = await api.getContentItem({ contentID: 22, locale: 'en-us' })
    console.log(item)
} catch (error) {
    console.error(error)
}

// Promise.all — fetch multiple items in parallel (much faster than sequential)
const [item1, item2, item3] = await Promise.all([
    api.getContentItem({ contentID: 1, locale: 'en-us' }),
    api.getContentItem({ contentID: 2, locale: 'en-us' }),
    api.getContentItem({ contentID: 3, locale: 'en-us' })
])
```

### Pattern 4: Validation Happens First

Every method validates BEFORE making an API call — fast failure with clear messages, not wasted network requests.

### Pattern 5: Configuration is Stored in Instance

Set once in `getApi()`, reused across all method calls automatically.

### Pattern 6: Error Handling is Gentle

404s return `undefined`. Server errors log a message and return `undefined`. Only network failures throw. This means you should always check for `undefined` before using a result.

---

## Part 9: Questions to Ask Yourself While Learning

### About getApi()
* What does getApi() return?
* What happens if you don't provide guid or apiKey?
* Can you call getApi() multiple times with different configs?

### About Methods
* Why does every method start with validation?
* What's the purpose of defaultParams?
* How does the URL get built from parameters?
* Why do methods use `this.makeRequest()`?

### About Configuration
* What's the difference between `isPreview: false` and `isPreview: true`?
* What does `logLevel: 'silent'` do?
* When would you set `requiresGuidInHeaders: true`?
* What happens if `maxAge` is set to 0?

### About Types
* What does `<T>` mean in `ContentItem<T>`?
* Why is `fields` in ContentItem generic?
* What is a `ContentZone` and how does it differ from a `ContentItem`?
* What does `state: number` in `ContentItemProperties` represent?

### About Data Flow
* How does data flow from Agility server to your app?
* What happens if the API returns unexpected data?
* How do you handle missing fields gracefully?
* Why does the SDK return `undefined` instead of throwing on 404?

---

## Part 10: Quick Reference - The Most Important Files

### `src/index.ts` — The Front Door
**Purpose:** Single export point. Only exports `getApi`. Everything else is hidden.

### `src/api-client.ts` — The Brain
**Purpose:** Initialize client, provide methods, make HTTP requests.

Three parts: `getApi()` function → `ApiClient` class with all 10 methods → `makeRequest()` which does the actual HTTP call.

### `src/methods/getContentItem.ts` — The Template Method
**Purpose:** Fetch a single content item. Understanding this file means understanding all 10 methods — same pattern, different URLs and validations.

### `src/utils.ts` — Shared Utilities
**Key functions:** `buildRequestUrlPath()` (builds base URL), `buildAuthHeader()` (adds API key to request headers), logging functions.

### `src/types/Config.ts` — Configuration Shape
What you pass to `getApi()`.

### `src/types/ContentItem.ts` — Data Shape
What you receive back. The `<T>` generic lets you type your custom `fields`.

### `src/types/Page.ts` — Page Structure
Pages contain `zones`, which contain `ContentItems`. This is the core page model in Agility CMS.

---

## Part 11: Installation Methods ⭐ NEW

There are three ways to install and use this SDK depending on your project setup.

### Method 1: npm (Recommended for most projects)

```bash
npm install @agility/content-fetch
```

```typescript
import agility from '@agility/content-fetch'

const api = agility.getApi({
    guid: 'your-guid',
    apiKey: 'your-api-key'
})
```

### Method 2: yarn

If your project uses Yarn as its package manager:

```bash
yarn add @agility/content-fetch
```

Usage is identical to npm — same import statement, same API. The only difference is how the package is installed and tracked (`yarn.lock` vs `package-lock.json`).

**When to use yarn vs npm:**
- Use `yarn` if your project already has a `yarn.lock` file
- Use `npm` if your project already has a `package-lock.json` file
- Never mix the two in the same project — pick one and stick with it

### Method 3: CDN / Script Tag (Browser only)

If you're not using a bundler (plain HTML/JS without npm), you can load the SDK directly in the browser via a `<script>` tag:

```html
<!-- Use a specific version (recommended for production) -->
<script type="text/javascript"
  src="https://unpkg.com/@agility/content-fetch@0.4.2/dist/agility-content-fetch.browser.js">
</script>

<!-- Or always use the latest version (easier but less stable) -->
<script type="text/javascript"
  src="https://unpkg.com/@agility/content-fetch@latest/dist/agility-content-fetch.browser.js">
</script>
```

After loading via script tag, the SDK is accessible as a **global variable** called `agility`:

```javascript
// No import statement needed!
const api = agility.getApi({
    guid: 'your-guid',
    apiKey: 'your-api-key'
})

api.getContentItem({ contentID: 22, locale: 'en-us' })
    .then(function(item) {
        console.log(item)
    })
    .catch(function(error) {
        console.error(error)
    })
```

**When to use the CDN approach:**
- Prototyping / quick demos
- Plain HTML projects without a build step
- Learning the SDK in a browser sandbox (CodePen, JSFiddle, etc.)

**⚠️ Caution:** The CDN approach exposes your API key in the browser's source code. Only use your **live** (read-only) API key this way — never your preview key or management key.

### Choosing the Right Installation

| Scenario | Method |
|---|---|
| Next.js, Remix, Gatsby, Vite | npm or yarn |
| Create React App | npm or yarn |
| Plain HTML + no bundler | CDN / script tag |
| Prototyping in CodeSandbox | CDN / script tag |

---

## Part 12: Contributing to the SDK ⭐ NEW

The Agility Content Fetch JS SDK is open source (MIT license). That means you can read the code, report issues, and even contribute fixes or improvements.

### How to Contribute

1. **Fork the repository** on GitHub at https://github.com/agility/agility-content-fetch-js-sdk
2. **Clone your fork** locally:
   ```bash
   git clone https://github.com/YOUR-USERNAME/agility-content-fetch-js-sdk.git
   cd agility-content-fetch-js-sdk
   ```
3. **Install dependencies:**
   ```bash
   npm install
   # or
   yarn install
   ```
4. **Create a branch** for your change:
   ```bash
   git checkout -b fix/my-bug-fix
   ```
5. **Make your changes** and add/update tests
6. **Run the test suite** to verify nothing is broken:
   ```bash
   npm run test
   ```
7. **Submit a Pull Request** on GitHub with a clear description of what you changed and why

### Running the Tests

Unit tests are a critical part of this SDK. Before any code merges, tests must pass. To run them:

```bash
npm run test
```

The test suite uses **Jest** (configured in `jest.config.js`) and **Mocha/Chai** for some older tests. Tests live in the `test/` folder and are organized by method:

```
test/
├── getContentItem.tests.js
├── getContentList.tests.js
├── getPage.tests.js
├── getUrlRedirections.tests.js
└── ... (one file per method)
```

**Why you should look at the tests even if you're not contributing:**

Reading tests is one of the fastest ways to understand how code is meant to be used. Each test file shows: what inputs are valid, what outputs are expected, and what edge cases matter.

### Reporting Issues

Found a bug? Open an issue at https://github.com/agility/agility-content-fetch-js-sdk/issues

When filing an issue, include:
- The SDK version you're using (`npm list @agility/content-fetch`)
- A minimal code example that reproduces the problem
- The expected vs. actual behavior

### License

The SDK is published under the **MIT License**, which means you can freely use it in personal and commercial projects, modify the code, and redistribute it — as long as you include the original license notice.

---

## Part 13: Content Webhooks ⭐ NEW

Webhooks let Agility CMS notify your application automatically whenever content is published or updated. This is what powers real-time content updates without you having to poll the API repeatedly.

### How Webhooks Work

```
Content Editor publishes in Agility CMS
           │
           ▼
Agility CMS sends HTTP POST to your webhook URL
           │
           ▼
Your app receives the notification
           │
           ▼
Your app re-fetches or revalidates affected content
           │
           ▼
Users see updated content
```

### Setting Up a Webhook in Next.js

A common pattern is to create a `/api/revalidate` endpoint in Next.js that Agility calls whenever content changes:

```typescript
// pages/api/revalidate.ts (Next.js Pages Router)
import type { NextApiRequest, NextApiResponse } from 'next'

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
    // Validate the webhook came from Agility
    const token = req.headers['x-agility-webhook-token']
    if (token !== process.env.AGILITY_WEBHOOK_SECRET) {
        return res.status(401).json({ message: 'Unauthorized' })
    }

    // Get the content that changed
    const { contentID, languageCode } = req.body

    try {
        // Revalidate the specific page(s) affected
        await res.revalidate(`/blog/${contentID}`)
        await res.revalidate('/')  // Also revalidate homepage if it shows recent posts
        return res.json({ revalidated: true })
    } catch (error) {
        return res.status(500).json({ message: 'Error revalidating' })
    }
}
```

### What Events Can Trigger a Webhook?

Agility CMS can send webhooks for:
- **Publish events** — when content is published live
- **Save events** — when content is saved (even unpublished drafts)
- **Workflow events** — when content moves through an approval workflow

### Where to Configure Webhooks

Webhooks are configured inside your Agility CMS instance settings, not in the SDK. You provide:
1. The URL Agility should POST to (your endpoint)
2. Which events should trigger it (publish, save, workflow)
3. A secret token for verification (optional but recommended)

See the official guide: https://help.agilitycms.com/hc/en-us/articles/360035934911

---

## Part 14: Official Tutorials & Resources ⭐ NEW

These are the official learning resources from Agility CMS. Use these alongside this guide.

### Tutorials (from the official README)

| Title | What You'll Learn |
|---|---|
| [About the Content Fetch API](https://help.agilitycms.com/hc/en-us/articles/360031985112) | High-level overview of the Fetch API and when to use it |
| [Authenticating your Content Fetch API Calls](https://help.agilitycms.com/hc/en-us/articles/360032225191) | How guid and apiKey work, live vs. preview keys |
| [Retrieving your API Key(s), Guid, and API URL](https://help.agilitycms.com/hc/en-us/articles/360031919212) | Step-by-step: finding your credentials in the Agility CMS dashboard |
| [Making your First Call with the Content Fetch API](https://help.agilitycms.com/hc/en-us/articles/360031918152) | Your first working API request |
| [Calling the Content Fetch API using the JS SDK](https://help.agilitycms.com/hc/en-us/articles/360031945912) | SDK-specific walkthrough |
| [Page Management in a Headless CMS](https://help.agilitycms.com/hc/en-us/articles/360032554331) | How pages, zones, and modules work together |
| [Using Agility CMS with Create React App](https://help.agilitycms.com/hc/en-us/articles/360031121692) | Build a CRA app with Agility content |
| [Creating a Module for the Agility CMS CRA](https://help.agilitycms.com/hc/en-us/articles/360031590791) | How to build reusable content modules |
| [Creating a Page Template for the CRA](https://help.agilitycms.com/hc/en-us/articles/360032611011) | How page templates work |
| [Deploying your Agility CMS Create React App](https://help.agilitycms.com/hc/en-us/articles/360032203552) | Deploying to production |
| [Content Webhooks](https://help.agilitycms.com/hc/en-us/articles/360035934911) | Setting up real-time content notifications |

### Recommended Learning Order for Beginners

1. Start with **"About the Content Fetch API"** — understand the big picture
2. Then **"Authenticating your Content Fetch API Calls"** — get your credentials
3. Then **"Making your First Call"** — get something working
4. Then **"Using the JS SDK"** — switch from raw fetch to this SDK
5. Then **"Page Management in a Headless CMS"** — understand zones/modules
6. Finally pick the framework tutorial that matches your project (React, Next.js, etc.)

### Official Package Links

| Resource | URL |
|---|---|
| GitHub Repository | https://github.com/agility/agility-content-fetch-js-sdk |
| npm Package | https://www.npmjs.com/package/@agility/content-fetch |
| Agility CMS Help Center | https://help.agilitycms.com/hc/en-us |
| Free Trial Sign-up | https://agilitycms.com/free |

---

## Part 15: SDK Versioning & Releases ⭐ NEW

### Current Version

The SDK follows [Semantic Versioning](https://semver.org/) (semver): `MAJOR.MINOR.PATCH`

- **MAJOR** — Breaking changes (rare). Existing code may need updates.
- **MINOR** — New features added in a backwards-compatible way.
- **PATCH** — Bug fixes only. Safe to update without any code changes.

As of early 2026, the SDK is at **v2.x**. You can see all releases at https://github.com/agility/agility-content-fetch-js-sdk/releases

### How to Check Your Installed Version

```bash
npm list @agility/content-fetch
# or
cat node_modules/@agility/content-fetch/package.json | grep '"version"'
```

### How to Update

```bash
# Update to latest
npm install @agility/content-fetch@latest

# Update to a specific version
npm install @agility/content-fetch@2.0.10
```

### Notable Version History

- **v2.x** — Current stable branch. Uses native `fetch()` API (built into modern Node.js 18+). Added East US region support in v2.0.10.
- **v1.x** — Used `axios` as the HTTP client (you may see references to this in older tutorials or the package.json `dependencies` field).

> 💡 **Why does this matter?** If you read code examples online that mention `axios` in the context of this SDK, they were written for v1.x. The v2.x SDK replaced axios with the native `fetch()` API for smaller bundle size and better compatibility with modern runtimes like Node 18+, Deno, and edge functions.

### Staying Up to Date

The SDK repository uses GitHub Actions for CI/CD. You can watch the repo on GitHub to get notified of new releases. It is generally safe to update to new PATCH and MINOR versions without changing your code.

---

## Summary: The Big Picture

### What is the SDK?

A JavaScript/TypeScript library that makes it easy to read content from Agility CMS. It works in both Node.js and browser environments, can be installed via npm, yarn, or a CDN script tag, and is fully open source under the MIT license.

### How does it work?

1. **Install:** `npm install @agility/content-fetch`
2. **Initialize:** `const api = agility.getApi({ guid, apiKey })`
3. **Call methods:** `api.getContentItem({ contentID, locale })`
4. **Methods do:** Validate → build URL → call makeRequest()
5. **makeRequest() does:** HTTP GET → parse JSON → return typed data
6. **You get:** A Promise with typed data (ContentItem, Page, etc.)
7. **Webhooks (optional):** Agility POSTs to your endpoint when content changes → you revalidate

### Directory Structure

```
src/
├── index.ts              ← Entry point (exports getApi only)
├── api-client.ts         ← Main client class
├── utils.ts              ← Helper functions
├── methods/              ← 10 methods (understand one = understand all)
└── types/                ← TypeScript definitions
.github/workflows/        ← CI/CD pipelines (auto-test on every push)
test/                     ← Unit tests (one file per method)
```

### Key Files in Order (50 minutes to 80% understanding)

1. `src/index.ts` (5 min)
2. `src/api-client.ts` (20 min)
3. `src/methods/getContentItem.ts` (10 min)
4. `src/types/Config.ts` (10 min)
5. `src/types/ContentItem.ts` (5 min)

### Learning Timeline

| Phase | Days | Focus |
|---|---|---|
| Foundation | 1-2 | What is the SDK? How does headless CMS work? |
| Core Files | 3-4 | `index.ts`, `api-client.ts`, `utils.ts` |
| Methods | 5-7 | All 10 methods, filtering, pagination |
| Types | 8-9 | TypeScript generics, ContentItem, Page, zones |
| Practice | 10+ | Build something real! |

### Final Tips

1. **Start simple** — Get a single item before filtering lists
2. **Use TypeScript** — Get autocompletion and type safety
3. **Log everything initially** — Set `logLevel: 'debug'` while learning
4. **Read the tests** — `test/` folder shows exactly how each method is meant to behave
5. **Use async/await** — Cleaner than `.then()`/`.catch()`
6. **Check for undefined** — API methods return `undefined` on 404, always guard against this
7. **Cache when appropriate** — Improves SSR performance
8. **Use environment variables** — Never hardcode guid/apiKey in source code
9. **Try preview mode** — Essential for content editing workflows
10. **Follow webhooks for live updates** — Avoid polling the API

---

## Additional Resources

| Resource | URL |
|---|---|
| GitHub Repository | https://github.com/agility/agility-content-fetch-js-sdk |
| npm Package | https://www.npmjs.com/package/@agility/content-fetch |
| Agility CMS | https://agilitycms.com |
| Agility Help Center | https://help.agilitycms.com/hc/en-us |
| Free Trial | https://agilitycms.com/free |
| SDK Reference Docs | https://agilitydocs.netlify.app/agility-content-fetch-js-sdk/ |
| Content Webhooks Guide | https://help.agilitycms.com/hc/en-us/articles/360035934911 |

---

## Glossary of Terms

**API** — Application Programming Interface. Software that lets apps communicate.

**Axios** — An HTTP client library used in SDK v1.x. Replaced by native `fetch()` in v2.x.

**CDN** — Content Delivery Network. Servers distributed globally to serve files fast.

**CI/CD** — Continuous Integration / Continuous Deployment. Automated pipelines that run tests and deploy code automatically.

**Content Item** — A single piece of content (blog post, product, person, etc.)

**Content Type** — Definition of what fields a content item has (like a blueprint)

**Content Reference** — A link from one content item to another

**Content Zone** — Area on a page where content modules are placed

**Fetch API** — Agility's read-only API for getting content. (Not write/modify)

**Fork** — A personal copy of an open source repository on GitHub, used for contributing.

**Generic** — Template parameter in TypeScript (like `<T>`)

**Headless CMS** — Content management system without a built-in frontend. Your app provides the "head."

**HTTP Request** — Message sent to a web server (GET, POST, etc.)

**Interface** — TypeScript definition of an object's shape

**Locale** — Language and region code (en-us, fr-ca, etc.)

**MIT License** — Open source license allowing free commercial use, modification, and distribution.

**npm** — Node Package Manager. The most common way to install JavaScript packages.

**Page** — Web page made up of zones containing modules

**Preview Mode** — A special mode that shows unpublished/draft content

**Promise** — JavaScript object representing eventual completion of an async operation

**Pull Request (PR)** — A request to merge your changes into a project on GitHub

**Semver** — Semantic Versioning. A version numbering convention: MAJOR.MINOR.PATCH

**Sitemap** — List of all pages/URLs on the site

**Sync** — Keeping your app's data in sync with Agility (see `@agility/content-sync`)

**TypeScript** — JavaScript with type safety

**Webhook** — An HTTP callback that notifies your app when something happens in the CMS

**yarn** — An alternative to npm for installing JavaScript packages

---

## FAQ

**Q: How long does it take to learn this SDK?**
A: 10-14 days at 1-2 hours/day for a complete beginner. 3-5 days if you're already familiar with REST APIs.

**Q: Do I need to know TypeScript?**
A: No — you can use it with plain JavaScript. But TypeScript gives you autocomplete and type safety, which makes learning much easier.

**Q: Should I use npm or yarn?**
A: Either works. Use whichever your project already uses. Don't mix them.

**Q: What's the difference between getContentItem and getPage?**
A: A ContentItem is a standalone piece of content. A Page is a web page that contains multiple content items arranged in zones.

**Q: Can I use this in the browser?**
A: Yes! Works in both Node.js and browser environments. For browser without a bundler, use the CDN / script tag method.

**Q: Is caching recommended?**
A: Yes for SSR apps. Start with `maxAge: 180000` (3 minutes). Disable it during development.

**Q: What's the difference between v1.x and v2.x?**
A: v1.x used `axios` as the HTTP client. v2.x uses the native `fetch()` API, which is built into Node.js 18+ and all modern browsers. The API surface (how you use it) is essentially the same.

**Q: What if my API key leaks?**
A: Rotate it immediately in your Agility CMS dashboard. Always use environment variables and never commit keys to git.

**Q: How do I know what content fields are available?**
A: Check your Agility CMS instance's content model definitions, or use TypeScript generics to type the `fields` object.

**Q: Can I contribute to this SDK?**
A: Yes! Fork the repo, make changes, run tests (`npm run test`), and open a Pull Request.

**Q: What happens when content changes in Agility CMS?**
A: By default, nothing until your next API call or server restart. To get real-time updates, set up a Content Webhook that triggers revalidation in your app.

---

**Last Updated:** 2026-03-04
**Guide Version:** 2.0

**Happy Learning! 🚀**
