# 🎓 Complete Beginner's Guide to Agility Content Fetch JS SDK

**Author:** GitHub Copilot  
**Created:** 2026-03-04  
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

---

## Part 1: What is this SDK and Why Does It Exist?

The **Agility Content Fetch JS SDK** is a JavaScript/TypeScript library that acts as a **bridge between your web application and Agility CMS**. Think of it as a translator:

```
Your Web App → (Fetch SDK) → Agility CMS Server → Gets Content Data → Returns to App
```

### Real-world Analogy

If Agility CMS is a restaurant kitchen storing recipes and ingredients, this SDK is like a waiter who knows:
- Exactly which kitchen door to knock on
- What to ask for
- How to bring the food back to your table in the right format

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
│       ├─�� ContentZone.ts
│       ├── FilterOperator.ts
│       └── ... (other types)
├── test/                         # Unit tests
├── package.json                  # Dependencies & scripts
├── README.md                     # Official documentation
├── tsconfig.json                 # TypeScript configuration
├── jest.config.js                # Test configuration
└── build.js                      # Build script
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
    // Throws error if contentID or locale missing
    
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
- `ContentItem.ts` - A piece of content
- `Page.ts` - A web page
- `ContentZone.ts` - Area on a page where content goes
- `Gallery.ts` - Collection of images
- `Config.ts` - Configuration options
- `FilterOperator.ts` - How to filter results

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
└────────────────┬──────────���─────────┘
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

### 2. Request Parameters

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
- `contentID`: Which piece of content to get
- `locale`: Which language/region (en-us, fr-ca, etc)
- `contentLinkDepth`: How many levels deep to fetch linked content
- `expandAllContentLinks`: Expand all references, even complex ones?

### 3. Response Types

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

### 4. Error Handling

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

### 5. Caching

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

---

## Part 5: Available Methods (What You Can Do)

### Overview Table

| Method | Purpose | Returns | When to Use |
|--------|---------|---------|------------|
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
- `eq` - equals
- `ne` - not equals
- `lt` - less than
- `lte` - less than or equal
- `gt` - greater than
- `gte` - greater than or equal
- `like` - contains (string)
- `in` - matches any in list
- `contains` - contains (for arrays)
- `range` - between two values

---

#### **5. getSitemapFlat() - Get All Pages as Flat List**

```typescript
const sitemap = await api.getSitemapFlat({
    channelName: 'website',             // REQUIRED
    locale: 'en-us'                     // REQUIRED
})

// Returns:
{
    pageID: 1,
    items: {
        '/': { pageID: 1, name: 'home', path: '/', ... },
        '/about': { pageID: 2, name: 'about', path: '/about', ... },
        '/contact': { pageID: 3, name: 'contact', path: '/contact', ... },
        '/blog': { pageID: 4, name: 'blog', path: '/blog', ... }
    }
}
```

**Use when:** You need page routing/navigation

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
            },
            {
                pageID: 11,
                path: '/blog/post-2',
                title: 'My Second Post',
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
        },
        // ... more images
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
console.log(syncResponse.items)  // Array of changed items
console.log(syncResponse.syncToken)  // Token for next call

// Continue syncing
syncResponse = await api.getSyncContent({
    syncToken: syncResponse.syncToken,  // Use returned token
    locale: 'en-us'
})

// Loop until syncToken is empty (you're caught up)
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

// Keep calling with returned syncToken until it's empty
```

**Use when:** You sync pages separately

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

**Use when:** You need to handle redirects

---

## Part 6: Learning Plan (Step by Step)

### Phase 1: Foundation (Days 1-2)
**Goal:** Understand the basics
**Time:** 2-3 hours total

**Tasks:**
- [ ] Read the README.md in the repo
- [ ] Understand what Agility CMS is (headless content management)
- [ ] Understand what "fetch API" means (retrieve/read content)
- [ ] Set up the SDK in a test project
  ```bash
  npm install @agility/content-fetch
  ```
- [ ] Create your first API client:
  ```javascript
  import agility from '@agility/content-fetch'
  const api = agility.getApi({
      guid: 'your-guid',
      apiKey: 'your-api-key'
  })
  ```

**Resources:**
- Official README: https://github.com/agility/agility-content-fetch-js-sdk
- Agility docs: https://help.agilitycms.com/hc/en-us
- About Headless CMS: https://www.agilitycms.com/resources/agility-headless-cms

**Self-Check:**
- Can you explain what the SDK does in one sentence?
- Can you set up the SDK with your credentials?

---

### Phase 2: Core Files (Days 3-4)
**Goal:** Understand code structure
**Time:** 2-3 hours

**Read these files in order:**

1. **`src/index.ts`** (5 min)
   - Question: Why is this file so small?
   - Answer: Hides complexity, exports one function

2. **`src/types/Config.ts`** (10 min)
   - Question: What configuration options are required?
   - Answer: guid and apiKey

3. **`src/api-client.ts`** (20 min)
   - Focus on lines 1-76 first (getApi function)
   - Then read lines 135-160 (method definitions)
   - Don't worry about makeRequest yet

4. **`src/utils.ts`** (15 min)
   - Question: What does buildAuthHeader do?
   - Question: What does buildRequestUrlPath do?

**Exercise:**
Draw a diagram showing:
- How index.ts connects to api-client.ts
- How api-client.ts uses utils.ts
- What methods are available on ApiClient

**Self-Check:**
- Can you explain what getApi() does?
- Can you explain what makeRequest() is?
- Can you explain what buildAuthHeader() does?

---

### Phase 3: Methods (Days 5-7)
**Goal:** Understand how to fetch data
**Time:** 3-4 hours

**Pick these methods in order:**

1. **`methods/getContentItem.ts`** (10 min)
   - Simplest method
   - Focus on: validate → build → request

2. **`methods/getPage.ts`** (10 min)
   - Similar to getContentItem
   - Different endpoint

3. **`methods/getContentList.ts`** (15 min)
   - More complex - has filtering
   - New concepts: skip, take, sort, filters

4. **`methods/getSitemapFlat.ts`** (10 min)
   - Different use case
   - Simpler parameters

**For each method ask yourself:**
- What parameters does it take?
- What does it validate?
- What URL does it build?
- What does it return?

**Exercise:**
Write pseudocode for a new method `getAuthorBio()` that:
- Takes authorID (required)
- Takes locale (required)
- Validates both
- Builds URL: `/author/${authorID}`
- Returns a Promise

**Self-Check:**
- Can you explain the flow of getContentItem?
- Can you explain what validation does?
- Can you explain what parameters mean?

---

### Phase 4: Type System (Days 8-9)
**Goal:** Understand data structures
**Time:** 2-3 hours

**Study these type files:**

1. **`types/Config.ts`** (10 min)
   - Already read, reinforce

2. **`types/ContentItem.ts`** (10 min)
   - Question: What does `<T = { [key: string]: any }>` mean?
   - Answer: Generic type for custom fields

3. **`types/Page.ts`** (15 min)
   - Focus on zones property
   - Question: What are zones?
   - Answer: Areas on page where content goes

4. **`types/ContentZone.ts`** (10 min)
   - Question: What's the difference between ContentItem and ContentZone?
   - Answer: Zone is placement of item on page

5. **`types/FilterOperator.ts`** (10 min)
   - Question: What operators are available?
   - Answer: eq, ne, lt, gt, lte, gte, like, in, contains, range

**Exercise:**
Create a TypeScript interface for your own content type:
```typescript
interface BlogPost {
    title: string;
    body: string;
    author: string;
    publishDate: Date;
    tags: string[];
    // ... add 5 more fields
}
```

Then use it:
```typescript
const post = await api.getContentItem<BlogPost>({
    contentID: 22,
    locale: 'en-us'
})

console.log(post.fields.title)  // TypeScript knows it's a string!
console.log(post.fields.tags)   // TypeScript knows it's string[]!
```

**Self-Check:**
- Can you explain what a ContentZone is?
- Can you explain generics (`<T>`)?
- Can you explain the difference between ContentItem and Page?

---

### Phase 5: Hands-On Practice (Days 10+)
**Goal:** Use the SDK in a real project
**Time:** 5-10 hours

**Build a simple React/Next.js app:**

```typescript
// 1. Initialize API
import agility from '@agility/content-fetch'

const api = agility.getApi({
    guid: process.env.REACT_APP_GUID,
    apiKey: process.env.REACT_APP_API_KEY
})

// 2. Fetch single content item
export async function getPost(postId: number) {
    return api.getContentItem({
        contentID: postId,
        locale: 'en-us'
    })
}

// 3. Fetch list with filtering
export async function getPosts(page: number = 1) {
    return api.getContentList({
        referenceName: 'blog_posts',
        locale: 'en-us',
        take: 10,
        skip: (page - 1) * 10,
        sort: 'properties.modified',
        direction: 'DESC'
    })
}

// 4. Fetch page with zones
export async function getHomePage() {
    return api.getPageByPath({
        pagePath: '/',
        channelName: 'website',
        locale: 'en-us'
    })
}

// 5. Display in React
function HomePage({ page }) {
    return (
        <div>
            <h1>{page.title}</h1>
            {page.zones.hero.map(zone => (
                <div key={zone.item.contentID}>
                    {zone.item.fields.title}
                </div>
            ))}
        </div>
    )
}
```

**Milestones:**
- ✅ Create API client
- ✅ Fetch single content item
- ✅ Fetch list of items
- ✅ Filter the list (by status, author, etc)
- ✅ Get a page with its zones
- ✅ Display all data in UI
- ✅ Add loading states
- ✅ Add error handling

**Self-Check:**
- Can you fetch data successfully?
- Can you display data in React?
- Can you filter/paginate?
- Can you handle errors?

---

## Part 7: Key Files to Study (In Priority Order)

### Priority 1 - MUST READ (2-3 hours total)

| File | Time | Why Important | Key Concept |
|------|------|---------------|------------|
| `README.md` | 10 min | Overview | What the SDK is |
| `src/index.ts` | 5 min | Entry point | Only exports getApi() |
| `src/api-client.ts` (lines 1-76) | 20 min | Initialization | How getApi() works |
| `src/api-client.ts` (lines 135-160) | 15 min | Methods | All available methods |
| `src/types/Config.ts` | 10 min | Configuration | What to pass to getApi() |
| `src/methods/getContentItem.ts` | 10 min | Simple example | How methods work |

**Total: ~70 minutes**

### Priority 2 - IMPORTANT (2-3 hours total)

| File | Time | Why Important | Key Concept |
|------|------|---------------|------------|
| `src/methods/getPage.ts` | 10 min | Page fetching | Similar pattern |
| `src/methods/getContentList.ts` | 15 min | Filtering | Most complex |
| `src/utils.ts` | 15 min | Helpers | URL & header building |
| `src/types/Page.ts` | 10 min | Page structure | Zones concept |
| `src/types/ContentItem.ts` | 10 min | Item structure | Generic types |
| `src/types/ContentZone.ts` | 5 min | Zone structure | Content placement |

**Total: ~65 minutes**

### Priority 3 - REFERENCE (1-2 hours total)

| File | Time | When Needed |
|------|------|-------------|
| `src/methods/getSitemapFlat.ts` | 5 min | Building routing |
| `src/methods/getSitemapNested.ts` | 5 min | Building menus |
| `src/methods/getGallery.ts` | 5 min | Displaying media |
| `src/methods/getSyncContent.ts` | 10 min | Syncing content |
| `src/types/FilterOperator.ts` | 5 min | Advanced filtering |
| `src/types/Client.ts` | 5 min | Type definitions |

**Total: ~35 minutes**

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
const defaultParams = {
    optionalParam: 10
}

// 3. Implement the method
function getSomething(
    this: ApiClientInstance,
    requestParams: SomethingRequestParams
): Promise<Something> {
    
    // STEP A: Validate
    validateRequestParams(requestParams);
    
    // STEP B: Apply defaults
    requestParams = { ...defaultParams, ...requestParams };
    
    // STEP C: Build request
    const req = {
        url: `...`,
        method: 'get',
        baseURL: buildRequestUrlPath(...),
        headers: buildAuthHeader(...),
        params: {}
    };
    
    // STEP D: Make request
    return this.makeRequest(req);
}

// 4. Validation function
function validateRequestParams(requestParams) {
    if (!requestParams.requiredParam) {
        throw new TypeError('Required param missing');
    }
}

// 5. Export
export default getSomething;
```

**This repeats for every method.** Understanding one method = understanding all of them!

---

### Pattern 2: Parameters Build the URL

Every API request follows this URL structure:

```
Base + Endpoint + Query String

Examples:
https://api.agilitycms.com/fetch/en-us/item/22?contentLinkDepth=1&expandAllContentLinks=false
                        │      │      │   │                    └─ contentLinkDepth param
                        │      │      │   └────── contentID param
                        │      │      └────────── locale parameter
                        │      └────────────────── fetch (not preview)
                        └────────────────────────── baseUrl

https://api.agilitycms.com/fetch/en-us/page/1?contentLinkDepth=2&expandAllContentLinks=true
https://api.agilitycms.com/fetch/en-us/contentlist/blog_posts?take=10&skip=0&sort=properties.modified
https://api.agilitycms.com/fetch/en-us/sitemap/flat/website
```

**How it's built:**
```typescript
// Method receives parameters
{ contentID: 22, locale: 'en-us' }

// buildRequestUrlPath() creates base
baseURL = 'https://api.agilitycms.com/fetch/en-us'

// Method creates endpoint
url = '/item/22?contentLinkDepth=1&...'

// Combined in makeRequest():
fullUrl = baseURL + url
        = 'https://api.agilitycms.com/fetch/en-us/item/22?...'
```

---

### Pattern 3: Promises for Async Operations

All methods return Promises. Use either style:

```typescript
// Style 1: .then/.catch
api.getContentItem({ contentID: 22, locale: 'en-us' })
    .then(item => {
        console.log(item);
    })
    .catch(error => {
        console.error(error);
    })

// Style 2: async/await (cleaner!)
try {
    const item = await api.getContentItem({
        contentID: 22,
        locale: 'en-us'
    })
    console.log(item);
} catch (error) {
    console.error(error);
}

// Style 3: Promise.all (multiple requests)
const [item1, item2, item3] = await Promise.all([
    api.getContentItem({ contentID: 1, locale: 'en-us' }),
    api.getContentItem({ contentID: 2, locale: 'en-us' }),
    api.getContentItem({ contentID: 3, locale: 'en-us' })
])
```

---

### Pattern 4: Validation Happens First

Every method validates BEFORE making API call:

```typescript
// BAD: Would make API request and fail
api.getContentItem({
    locale: 'en-us'
    // Missing contentID!
})

// GOOD: Fails immediately with clear error
// Error: "You must include a contentID number in your request params."

// WHY: Validation prevents wasted API calls
// Benefits:
// - Faster feedback
// - Clearer error messages
// - Saves API quota
```

---

### Pattern 5: Configuration is Stored in Instance

```typescript
// Configuration is set once
const api = agility.getApi({
    guid: 'abc123',
    apiKey: 'secret',
    logLevel: 'warn',
    caching: { maxAge: 180000 }
})

// All methods reuse same config
api.getContentItem({...})  // Uses config
api.getContentList({...})  // Uses config
api.getPage({...})         // Uses config

// No need to pass config to each method
```

---

### Pattern 6: Error Handling is Gentle

```typescript
// 404 Not Found
const item = await api.getContentItem({
    contentID: 999,  // doesn't exist
    locale: 'en-us'
})
console.log(item)  // undefined (not an error)

// 500 Server Error
const item = await api.getContentItem({...})
// Logs error, returns undefined

// Network Error
const item = await api.getContentItem({...})
// Throws exception (not caught internally)

// Parameter Error
try {
    const item = await api.getContentItem({
        // Missing locale
    })
} catch (e) {
    console.log(e.message)  // "You must include a locale..."
}
```

---

## Part 9: Questions to Ask Yourself While Learning

As you read the code, ask these questions:

### About getApi()
- [ ] What does getApi() do?
- [ ] What does it return?
- [ ] What happens to the config object?
- [ ] Why validate the config immediately?
- [ ] What happens if you don't provide guid or apiKey?

### About Methods
- [ ] Why does every method start with validation?
- [ ] What's the purpose of defaultParams?
- [ ] How does the URL get built?
- [ ] What are baseURL and url and why are they separate?
- [ ] Why do methods use `this.makeRequest()`?

### About Configuration
- [ ] What's the difference between baseUrl and apiKey?
- [ ] When would you use isPreview=true?
- [ ] What does logLevel control?
- [ ] How does caching work?
- [ ] Why is guid sometimes in headers and sometimes not?

### About Types
- [ ] What's a TypeScript interface?
- [ ] Why use interfaces instead of just JavaScript objects?
- [ ] What does `<T>` mean in `ContentItem<T>`?
- [ ] Why is fields in ContentItem generic?
- [ ] How do you know what fields are available?

### About Utils
- [ ] What does buildAuthHeader() do?
- [ ] What does buildRequestUrlPath() do?
- [ ] Why check isHttps()?
- [ ] What do the logging functions do?
- [ ] When would you use each log level?

### About Data Flow
- [ ] How does data flow from Agility server to my app?
- [ ] Where does parsing happen?
- [ ] Where does type checking happen?
- [ ] What happens if API returns unexpected data?
- [ ] How do you handle missing fields?

---

## Part 10: Quick Reference - The Most Important Files

### `src/index.ts` - The Front Door

**Size:** 28 lines  
**Purpose:** Export public API  
**Read time:** 5 minutes

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

**Key takeaway:** Only one thing is exported: `getApi`. Everything else is hidden.

---

### `src/api-client.ts` - The Brain

**Size:** ~300 lines  
**Purpose:** Initialize client, provide methods, make requests  
**Read time:** 40 minutes

**Three main parts:**

**Part A: The getApi() function (lines ~135-160)**
```typescript
export function getApi(config: Config) {
    validateConfigParams(config);
    if (!config.fetchConfig) config.fetchConfig = {}
    return new ApiClient(config);
}
```
Takes config, validates, returns client.

**Part B: The ApiClient class (lines ~80-250)**
```typescript
class ApiClient {
    config: Config;
    
    async getContentItem(params) { ... }
    async getContentList(params) { ... }
    async getPage(params) { ... }
    // ... and 7 more methods
    
    async makeRequest(reqConfig) {
        // Does actual fetch() call
    }
}
```
Stores config, provides all methods.

**Part C: The makeRequest() method (lines ~200-250)**
```typescript
async makeRequest(reqConfig: RequestParams) {
    const fullUrl = `${reqConfig.baseURL}${reqConfig.url}`
    
    const init = {
        method: "GET",
        headers: { ...reqConfig.headers },
    }
    
    const response = await fetch(fullUrl, init)
    let data = await response.json()
    return data
}
```
Does the actual HTTP request using fetch API.

**Key takeaway:** ApiClient is the main class. Every method delegates to methods/ folder, which calls makeRequest().

---

### `src/methods/getContentItem.ts` - Example Method

**Size:** 79 lines  
**Purpose:** Fetch single content item  
**Read time:** 10 minutes

**Pattern to understand:**
```typescript
// 1. Interface for parameters
export interface ContentItemRequestParams {
    contentID: number;
    locale?: string;
    contentLinkDepth?: number;
    expandAllContentLinks?: boolean;
}

// 2. Function
function getContentItem(this: ApiClientInstance, requestParams) {
    // Validate
    // Apply defaults
    // Build request
    // Call makeRequest()
}

// 3. Validation
function validateRequestParams(requestParams) {
    if (!requestParams.contentID) {
        throw new TypeError('...');
    }
}

// 4. Export
export default getContentItem;
```

**Key takeaway:** Understand this file = understand all 10 methods. Same pattern, just different URLs and validations.

---

### `src/utils.ts` - Helper Functions

**Size:** 122 lines  
**Purpose:** Shared utilities  
**Read time:** 15 minutes

**Key functions:**

```typescript
// Builds base URL path
buildRequestUrlPath(config, locale)
// Returns: https://api.agilitycms.com/fetch/en-us
// Or:      https://api.agilitycms.com/preview/en-us

// Builds authentication headers
buildAuthHeader(config)
// Returns: { APIKey: '...', Guid: '...' }

// Builds query string
buildPathUrl(contentType, referenceName, skip, take, ...)
// Returns: /contentlist/blog_posts?take=10&skip=0&...

// Logging functions
logDebug({ config, message })
logInfo({ config, message })
logWarning({ config, message })
logError({ config, message })
```

**Key takeaway:** These functions are used by every method to construct URLs and headers.

---

### `src/types/Config.ts` - Configuration

**Size:** 48 lines  
**Purpose:** Define configuration shape  
**Read time:** 10 minutes

```typescript
export interface Config {
    guid?: string;
    apiKey?: string;
    baseUrl?: string;
    isPreview?: boolean;
    locale?: string;
    headers?: { [key: string]: string };
    logLevel?: 'debug' | 'info' | 'warn' | 'error' | 'silent';
    caching?: { maxAge?: number };
    // ... more fields
}
```

**Key takeaway:** This defines what configuration options are available when you call getApi().

---

### `src/types/ContentItem.ts` - Data Shape

**Size:** 29 lines  
**Purpose:** Define content item structure  
**Read time:** 5 minutes

```typescript
export interface ContentItem<T = { [key: string]: any }> {
    contentID: number;
    properties: ContentItemProperties;
    fields: T;
    seo?: SEOProperties;
}
```

**Key takeaway:** This shows what data looks like when you fetch a content item. The `<T>` generic lets you type your custom fields.

---

### `src/types/Page.ts` - Page Structure

**Size:** 62 lines  
**Purpose:** Define page structure  
**Read time:** 10 minutes

```typescript
export interface Page {
    pageID: number;
    name: string;
    path: string;
    title: string;
    zones: { [key: string]: ContentZone[] };  // KEY CONCEPT
    properties: SystemProperties;
    // ... more fields
}
```

**Key takeaway:** Pages contain zones, which contain content. This is how pages are structured in Agility.

---

## Summary: The Big Picture

### What is the SDK?
A JavaScript library that makes it easy to fetch content from Agility CMS.

### How does it work?

1. **Initialize:** `const api = agility.getApi({ guid, apiKey })`
2. **Call methods:** `api.getContentItem({ contentID, locale })`
3. **Method does:**
   - Validate parameters
   - Build HTTP request
   - Call makeRequest()
4. **makeRequest() does:**
   - Uses fetch() API
   - Calls Agility servers
   - Parses response
   - Returns typed data
5. **You get:** Promise with typed data (ContentItem, Page, etc.)

### Directory Structure

```
src/
├── index.ts              ← Entry point
├── api-client.ts         ← Main client class
├── utils.ts              ← Helper functions
├── methods/              ← 10 methods for different requests
│   ├── getContentItem.ts (understand this = understand all)
│   ├── getPage.ts
│   ├── getContentList.ts (more complex)
│   └── ... 7 more ...
└── types/                ← TypeScript definitions
    ├── Config.ts         (what to pass to getApi)
    ├── ContentItem.ts    (what you get back)
    ├── Page.ts           (page structure with zones)
    └── ... 12 more ...
```

### Key Files in Order

**To understand 80% of the code:**
1. `src/index.ts` (5 min)
2. `src/api-client.ts` (20 min)
3. `src/methods/getContentItem.ts` (10 min)
4. `src/types/Config.ts` (10 min)
5. `src/types/ContentItem.ts` (5 min)

**Total: 50 minutes**

### Common Patterns

1. **All methods validate first** - fail fast
2. **All methods build URLs the same way** - understand once, apply everywhere
3. **All methods return Promises** - use async/await
4. **Configuration is set once** - stored in instance
5. **Error handling is gentle** - 404s return undefined, others log

### Learning Timeline

- **Days 1-2:** Foundation (what is SDK?)
- **Days 3-4:** Core files (how does it work?)
- **Days 5-7:** Methods (what can I do?)
- **Days 8-9:** Types (what data comes back?)
- **Days 10+:** Practice (build something!)

**Total time:** 10-14 days at 1-2 hours/day

### What You Should Know After Reading This Guide

- ✅ What the SDK does and why it exists
- ✅ How it's organized
- ✅ The main entry point (getApi)
- ✅ How methods work
- ✅ What each main file does
- ✅ How data flows
- ✅ What types are available
- ✅ How to use it in your app

---

## Additional Resources

### Official Documentation
- [Agility CMS Official Site](https://agilitycms.com)
- [GitHub Repository](https://github.com/agility/agility-content-fetch-js-sdk)
- [NPM Package](https://www.npmjs.com/package/@agility/content-fetch)
- [Agility Documentation](https://help.agilitycms.com/hc/en-us)

### Related Topics to Learn
- REST APIs and HTTP requests
- TypeScript basics and generics
- Promises and async/await
- Headless CMS concepts

### Community
- GitHub Issues: Report bugs or ask questions
- Agility Support: Official support channel

---

## Glossary of Terms

**API** - Application Programming Interface. Software that lets apps communicate.

**Fetch API** - Agility's read-only API for getting content. (Not write/modify)

**Headless CMS** - Content management system without a "head" (frontend). Your app is the head.

**Content Item** - A single piece of content (blog post, product, person, etc.)

**Content Type** - Definition of what fields a content item has (like a blueprint)

**Content Reference** - A link to another piece of content

**Content Zone** - Area on a page where content is placed

**Page** - Web page made up of zones

**Locale** - Language and region code (en-us, fr-ca, etc.)

**Sitemap** - List of all pages/URLs on the site

**Sync** - Keeping your app's data in sync with Agility

**TypeScript** - JavaScript with type safety

**Interface** - TypeScript definition of an object's shape

**Generic** - Template parameter in TypeScript (like `<T>`)

**Promise** - JavaScript object that represents eventual completion of async operation

**async/await** - Modern way to work with Promises

**HTTP Request** - Message sent to a web server (GET, POST, etc.)

**Headers** - Metadata included with HTTP requests

**Query String** - Parameters in URL after `?` (e.g., `?id=22&name=test`)

**Method** - Function available on an object (like `api.getContentItem()`)

---

## FAQ

### Q: How long does it take to learn this SDK?
A: For a beginner, 10-14 days of 1-2 hours per day. For someone familiar with APIs, 3-5 days.

### Q: Do I need to know TypeScript?
A: No, but it helps. You can use it with JavaScript too.

### Q: What's the difference between getContentItem and getPage?
A: ContentItem is a standalone piece of content. Page is a web page that may contain multiple ContentItems in zones.

### Q: Can I use this in the browser?
A: Yes! It works in both Node.js and browser environments.

### Q: Is caching recommended?
A: Yes, for better performance. Set `maxAge: 180000` (3 minutes) to start.

### Q: What if my API key leaks?
A: In production, use environment variables and never commit keys to git.

### Q: How do I handle pagination?
A: Use `take` and `skip` parameters in getContentList.

### Q: Can I filter content?
A: Yes, using the `filters` parameter in getContentList.

### Q: What if content doesn't exist?
A: SDK returns `undefined` for 404 responses.

### Q: How do I know what fields are available?
A: Check Agility CMS instance or use TypeScript generics to type them.

---

## Final Tips

1. **Start simple** - Get a single item before filtering lists
2. **Use TypeScript** - Get autocompletion and type safety
3. **Log everything initially** - Understand data flow
4. **Read error messages carefully** - They tell you exactly what's wrong
5. **Check the types** - They document what's available
6. **Use async/await** - Cleaner than .then()/.catch()
7. **Cache when appropriate** - Improves performance
8. **Validate user input** - Before sending to API
9. **Handle errors gracefully** - Log, show user-friendly message
10. **Read the official docs** - This guide is supplementary

---

**Last Updated:** 2026-03-04  
**Guide Version:** 1.0  
**Created by:** GitHub Copilot

**Happy Learning! 🚀**