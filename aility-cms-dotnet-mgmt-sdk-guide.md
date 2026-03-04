# 🎓 Complete Beginner's Guide to Agility CMS Management SDK for .NET

**Author:** Claude (Anthropic)
**Created:** 2026-03-04
**Difficulty Level:** Beginner
**Estimated Learning Time:** 10–14 days (1–2 hours per day)

---

## Table of Contents

1. [What is this SDK?](#part-1-what-is-this-sdk-and-why-does-it-exist)
2. [Fetch SDK vs. Management SDK — Key Differences](#part-2-fetch-sdk-vs-management-sdk--key-differences)
3. [Repository Structure Breakdown](#part-3-repository-structure-breakdown)
4. [How Everything Works Together](#part-4-how-everything-works-together)
5. [Installation](#part-5-installation)
6. [Authentication & Configuration](#part-6-authentication--configuration)
7. [Core Concepts](#part-7-core-concepts)
8. [Method Classes Overview](#part-8-method-classes-overview)
9. [ContentMethods — Managing Content Items](#part-9-contentmethods--managing-content-items)
10. [AssetMethods — Managing Files & Media](#part-10-assetmethods--managing-files--media)
11. [PageMethods — Managing Pages](#part-11-pagemethods--managing-pages)
12. [ModelMethods — Managing Content Models](#part-12-modelmethods--managing-content-models)
13. [ContainerMethods — Managing Containers](#part-13-containermethods--managing-containers)
14. [BatchMethods — Workflow & Publishing](#part-14-batchmethods--workflow--publishing)
15. [InstanceMethods & User Methods](#part-15-instancemethods--user-methods)
16. [Common Patterns You'll See](#part-16-common-patterns-youll-see)
17. [Error Handling](#part-17-error-handling)
18. [Learning Plan Step by Step](#part-18-learning-plan-step-by-step)
19. [Key Files to Study](#part-19-key-files-to-study)
20. [Questions to Ask Yourself While Learning](#part-20-questions-to-ask-yourself-while-learning)
21. [Quick Reference](#part-21-quick-reference)
22. [Contributing to the SDK](#part-22-contributing-to-the-sdk)
23. [Official Resources & Links](#part-23-official-resources--links)
24. [Glossary of Terms](#part-24-glossary-of-terms)
25. [FAQ](#part-25-faq)

---

## Part 1: What is this SDK and Why Does It Exist?

The **Agility CMS Management SDK for .NET** is a C# library that lets your .NET application **create, update, delete, and manage** content, pages, assets, models, and users in Agility CMS — all programmatically, without logging into the CMS interface.

Think of it like a remote control for your CMS:

```
Your .NET App → (Management SDK) → Agility CMS Management API → Modifies CMS Content
```

### Real-World Analogy

If Agility CMS is a restaurant:
- The **Fetch SDK** is a waiter who **brings food to your table** (reads content)
- The **Management SDK** is the **kitchen manager** who can **create new menu items, update recipes, and reorganize the entire kitchen** (writes, updates, deletes)

### What Problems Does It Solve?

**Without the SDK:** You'd have to manually:
- Log into the Agility CMS interface every time you want to create or update content
- Build complex REST API requests with custom authentication headers
- Handle error responses and retries yourself
- Write your own data serialization and deserialization

**With the SDK:**
```csharp
// Create a new blog post in 5 lines of C#
var contentItem = await clientInstance.contentMethods.SaveContentItem(
    new ContentItem { fields = new { title = "My Post", body = "Hello world" } },
    guid,
    locale
);
```

### What This SDK Is NOT

Understanding the boundaries matters:

- **NOT a read/fetch SDK.** It is for writing and managing content. To read published content in your web app, use the separate `@agility/content-fetch` (JS) or `agilitycms-dotnet-fetch-api` (.NET) packages.
- **NOT a sync SDK.** For locally caching content, use `agilitycms-dotnet-sync`.
- **NOT the CMS interface itself.** This SDK talks to the Management API behind the scenes — the same API that powers `app.agilitycms.com`.
- **NOT for anonymous/public use.** All requests require a **Bearer token** (authenticated access). Never expose management credentials publicly.

---

## Part 2: Fetch SDK vs. Management SDK — Key Differences

A common point of confusion for beginners is understanding which SDK to use and when. Here is a clear side-by-side comparison:

| Feature | Fetch SDK (`content-fetch`) | Management SDK (`management-sdk-dotnet`) |
|---|---|---|
| **Purpose** | Read published content | Create, update, delete content |
| **Authentication** | API Key (read-only) | Bearer Token (OAuth 2.0) |
| **Who uses it** | Your website / front-end app | Admin tools, CI/CD pipelines, migrations |
| **Operations** | GET only | GET + POST + PUT + DELETE |
| **Security risk** | Low (read-only key) | High (must protect Bearer token) |
| **Language** | JavaScript / TypeScript | C# / .NET |
| **Typical use case** | Displaying a blog post to a visitor | Importing 500 blog posts from a CSV file |

### When Would You Use the Management SDK?

- **Content migration** — Moving content from an old CMS into Agility in bulk
- **Automated content pipelines** — A script that creates content items from a data feed
- **CI/CD workflows** — Automatically publishing or approving content as part of a deployment
- **Custom admin tools** — Building an internal tool that lets your team manage content without logging into the Agility UI
- **Bulk operations** — Updating thousands of content items at once that would be tedious to do manually

---

## Part 3: Repository Structure Breakdown

```
agility-cms-management-sdk-dotnet/
├── management.api.sdk/               # Main SDK source code
│   ├── Methods/                      # All method classes (the toolbox)
│   │   ├── ContentMethods.cs         # Create, read, update, delete content items
│   │   ├── AssetMethods.cs           # Upload, move, delete files and media
│   │   ├── PageManageMethods.cs      # Create, update, delete pages
│   │   ├── ModelMethods.cs           # Manage content model definitions
│   │   ├── ContainerMethods.cs       # Manage content containers/lists
│   │   ├── BatchMethods.cs           # Publish, approve, decline batches
│   │   └── InstanceMethods.cs        # Instance-level operations (locales, users)
│   ├── Models/                       # C# model classes (data shapes)
│   │   ├── Options.cs                # SDK configuration (token, locale, guid)
│   │   ├── ContentItem.cs            # A piece of content
│   │   ├── Media.cs                  # A media/asset file
│   │   ├── AssetMediaList.cs         # List of media files
│   │   ├── AssetMediaGrouping.cs     # Gallery of media
│   │   ├── AssetGalleries.cs         # Collection of galleries
│   │   ├── AssetContainer.cs         # Media container
│   │   └── ... (other models)
│   ├── ClientInstance.cs             # The main client class — your entry point
│   └── management.api.sdk.csproj     # Project file
├── management.api.sdk.test/          # Unit test project
│   └── ... (test files)
└── agility-cms-management-sdk-dotnet.sln  # Visual Studio solution file
```

### Folder-by-Folder Breakdown

#### **1. `ClientInstance.cs` — The Front Door**

This is the first class you interact with. It exposes all the method classes as properties:

```csharp
ClientInstance clientInstance = new ClientInstance(options);

// Now you have access to all method groups:
clientInstance.contentMethods      // Content operations
clientInstance.assetMethods        // Asset/media operations
clientInstance.pageManageMethods   // Page operations
clientInstance.modelMethods        // Content model operations
clientInstance.containerMethods    // Container operations
clientInstance.batchMethods        // Batch/workflow operations
clientInstance.instanceMethods     // Instance-level operations
```

**Why one entry point?** It keeps your code clean. You create the client once, provide your credentials once, and all methods share the same authenticated connection.

---

#### **2. `Models/Options.cs` — The Configuration**

The `Options` class holds everything needed to authenticate and target the right instance:

```csharp
// The Options class holds your credentials
public class Options
{
    public string token { get; set; }   // Bearer token from OAuth login
    public string locale { get; set; }  // e.g. "en-us"
    public string guid { get; set; }    // Your Agility instance GUID
}
```

---

#### **3. `Methods/` — The Toolbox**

Each file in this folder handles one category of operations. Each class follows the same pattern:

1. Accept parameters (guid, locale, content data, etc.)
2. Build an authenticated HTTP request
3. Send to the Agility Management REST API
4. Return a typed C# object

---

#### **4. `Models/` — The Data Shapes**

C# classes that represent the data Agility returns or expects. These give you IntelliSense / autocomplete in Visual Studio or Rider and prevent runtime type errors.

---

#### **5. `management.api.sdk.test/` — The Tests**

Unit tests that verify each method works correctly. Useful for understanding expected inputs and outputs, even if you're not contributing.

---

## Part 4: How Everything Works Together

### The Complete Flow (Step by Step)

```
STEP 1: Configure
┌─────────────────────────────────────┐
│ Options options = new Options();    │
│ options.token = "Bearer eyJ...";    │
│ (from OAuth login)                  │
└────────────────┬────────────────────┘
                 │
                 ▼
STEP 2: Create Client
┌─────────────────────────────────────┐
│ ClientInstance client =             │
│   new ClientInstance(options);      │
│                                     │
│ Client stores options internally    │
│ Exposes all method group properties │
└────────────────┬────────────────────┘
                 │
                 ▼
STEP 3: Call a Method
┌─────────────────────────────────────┐
│ var item = await                    │
│   client.contentMethods             │
│          .GetContentItem(22,        │
│                          guid,      │
│                          locale);   │
└────────────────┬────────────────────┘
                 │
                 ▼
STEP 4: SDK Builds HTTP Request
┌─────────────────────────────────────┐
│ HTTP GET                            │
│ URL: https://mgmt.aglty.io/        │
│        {guid}/content/{locale}      │
│        /item/22                     │
│                                     │
│ Headers:                            │
│   Authorization: Bearer eyJ...      │
│   Content-Type: application/json   │
└────────────────┬────────────────────┘
                 │
                 ▼
STEP 5: Agility API Responds
┌─────────────────────────────────────┐
│ HTTP 200 OK                         │
│ Body: { contentID: 22,              │
│         fields: { ... } }           │
└────────────────┬────────────────────┘
                 │
                 ▼
STEP 6: SDK Deserializes Response
┌─────────────────────────────────────┐
│ JSON → ContentItem C# object        │
│ Returns typed object to your code   │
└─────────────────────────────────────┘
```

### Simplified Architecture Diagram

```
YOUR .NET APP
    │
    ├─→ new Options() { token, guid, locale }
    │
    ├─→ new ClientInstance(options)
    │         │
    │         ├── contentMethods    → Methods/ContentMethods.cs
    │         ├── assetMethods      → Methods/AssetMethods.cs
    │         ├── pageManageMethods → Methods/PageManageMethods.cs
    │         ├── modelMethods      → Methods/ModelMethods.cs
    │         ├── containerMethods  → Methods/ContainerMethods.cs
    │         └── batchMethods      → Methods/BatchMethods.cs
    │
    └─→ await client.contentMethods.GetContentItem(22, guid, locale)
              │
              └─→ HTTP Request to Agility Management API
                        │
                        └─→ Returns ContentItem C# object
```

---

## Part 5: Installation

### Option 1: Clone and Reference (Recommended for Learning)

The SDK is distributed as a Visual Studio solution you clone and reference directly in your project:

```bash
# 1. Clone the repository
git clone https://github.com/agility/agility-cms-management-sdk-dotnet.git

# 2. Open the solution in Visual Studio or Rider
# agility-cms-management-sdk-dotnet.sln

# 3. In your own project, add a project reference:
# Right-click Dependencies → Add Project Reference → management.api.sdk
```

### Option 2: NuGet Package

If a NuGet package is available for your version, install it via the CLI:

```bash
dotnet add package agility-cms-management-sdk-dotnet
```

Or via the Visual Studio NuGet Package Manager UI: search for `agility-cms-management-sdk-dotnet`.

### Prerequisites

Before using the SDK, you need:

1. **.NET SDK** installed on your machine (version 6.0 or later recommended)
   ```bash
   # Check your version
   dotnet --version
   ```

2. **An Agility CMS instance** — Sign up free at https://agilitycms.com/free

3. **A Bearer Token** — Obtained by authenticating via OAuth 2.0 (see Part 6)

4. **Your instance GUID** — Found in Agility CMS under Settings → API Keys

### Adding the Namespace

Once referenced, add the namespace at the top of your C# file:

```csharp
using management.api.sdk;
using agility.models;
```

---

## Part 6: Authentication & Configuration

This is the most important section for beginners, because the Management SDK requires **OAuth 2.0 Bearer Token authentication** — which is more involved than the simple API Key used by the Fetch SDK.

### Why Bearer Tokens Instead of API Keys?

The Management SDK can **create, update, and delete** content. That's powerful and potentially destructive. Bearer tokens:
- Expire after a set time (limiting damage if leaked)
- Are tied to a specific user's permissions
- Can be revoked instantly
- Support fine-grained access control

**Never use your management Bearer token in a public website or mobile app.** Keep it in environment variables or a secrets manager.

### Step 1: Initiate the authorization flow by making a GET request to the authorization endpoint:

```javascript
const authUrl = 'https://mgmt.aglty.io/oauth/authorize';

//if you wish to implement offline access using refresh tokens, use this URL (enables refresh tokens)
//const authUrl = 'https://mgmt.aglty.io/oauth/authorize?scope=offline-access '; 

const params = new URLSearchParams({
  response_type: 'code',
  redirect_uri: 'YOUR_REDIRECT_URI',
  state: 'YOUR_STATE',
  scope: 'openid profile email offline_access'
});

// Redirect the user to the authorization URL
window.location.href = `${authUrl}?${params.toString()}`;
```


#### Step 2: After successful authentication, you'll receive an authorization code at your redirect URI. Use this code to obtain an access token:

```javascript
const response = await fetch('https://mgmt.aglty.io/oauth/token', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded'
  },
  body: new URLSearchParams({
    code: 'YOUR_AUTHORIZATION_CODE'
  })
});

const { access_token, refresh_token, expires_in } = await response.json();
```

#### Step 3: When the access token expires, use the refresh token to obtain a new access token:

```javascript
const response = await fetch('https://mgmt.aglty.io/oauth/refresh', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    refresh_token: 'YOUR_REFRESH_TOKEN'
  })
});

const { access_token, refresh_token, expires_in } = await response.json();
```

#### Step 4: Use the obtained token to initialize the SDK:

```javascript
import * as mgmtApi from "@agility/management-sdk";

// Initialize the Options Class with your authentication token
let options = new mgmtApi.Options();
options.token = access_token; // Use the token obtained from authentication

// Initialize the APIClient Class
let apiClient = new mgmtApi.ApiClient(options);

let guid = "<<Provide the Guid of the Website>>";
let locale = "<<Provide the locale of the Website>>"; // Example: en-us

// Now you can make authenticated requests
var contentItem = await apiClient.contentMethods.getContentItem(22, guid, locale);
console.log(JSON.stringify(contentItem));
```


### Authentication with Personal Access Tokens (PAT)

Personal Access Tokens are an alternative to OAuth 2.0 for server-to-server and automation use cases. They skip the redirect flow — once generated, use the token directly without any OAuth steps.

**When to use PAT instead of OAuth 2.0:**
- CI/CD pipelines and automation scripts
- Server-side applications without an interactive user session
- Integration tooling where OAuth redirect flows aren't practical

#### Step 1: Generate a Personal Access Token

PATs are created via the Management API itself. You must first obtain an OAuth 2.0 access token (Steps 1–3 above), then call the token creation endpoint using the [Swagger docs](https://mgmt.aglty.io) for your region:

| Region | Swagger URL |
|---|---|
| US (default) | `https://mgmt.aglty.io` |
| Canada | `https://mgmt-ca.aglty.io` |
| Europe | `https://mgmt-eu.aglty.io` |
| Australia | `https://mgmt-aus.aglty.io` |
| Dev | `https://mgmt-dev.aglty.io` |

In Swagger, authenticate with your OAuth 2.0 access token, then call:

```
POST /api/v1/tokens/create
```

```json
{
  "name": "my-automation-token",
  "expiryDate": "2028-01-01T00:00:00Z"
}
```

> **Note:** Swagger defaults `expiryDate` to the current timestamp, which would make the token expire immediately. Update the year to a future date — tokens can be set up to 2 years from the creation date.

The response includes the token value — **copy it immediately, it will not be shown again**.


#### Step 2: Initialize the SDK

Pass the PAT directly as the `token` value. No OAuth flow or token exchange is required.

```csharp
using management.api.sdk;

var options = new agility.models.Options
{
    token = "YOUR_PERSONAL_ACCESS_TOKEN",
    locale = "en-us",
    guid = "your-website-guid"
};

var clientInstance = new ClientInstance(options);
```

The API automatically identifies PATs by their token signature and routes them through the appropriate authentication path. Your code does not need to specify the authentication type.

#### PAT Restrictions

PATs cannot access the following endpoints. Use OAuth 2.0 for these operations:

| Restricted Operation | Endpoints |
|---|---|
| User management | `POST/PUT/PATCH/DELETE /api/v*/instance/*/users` |
| Token management | `POST/PUT/DELETE/GET /api/v*/tokens/*` |
| Admin operations | `POST/PUT/DELETE /api/v*/admin/*` |

Requests to restricted endpoints with a PAT return `403 access_denied`.


---

## Part 7: Core Concepts

### 1. The Batch System

The most important concept unique to the Management SDK is **batches**. Understanding this will save you a lot of confusion.

In Agility CMS, content is never published instantly when you save it. Instead:

1. **You save content** → it gets added to a **Batch** (a queue of changes)
2. **You publish the Batch** → all changes in that batch go live at once

```
SaveContentItem() → Creates content + Returns BatchID
                              │
                              ▼
               Batch exists with status: Pending
                              │
              You call PublishBatch(batchID) 
                              │
                              ▼
               Content is now LIVE on the website
```

**Why batches?** They allow editors to:
- Group related changes together (e.g. all changes for a product launch)
- Review before publishing
- Approve/decline changes through a workflow
- Rollback an entire batch if something goes wrong

**As a developer**, when you call `SaveContentItem()`, the return value is a **BatchID** — not the ContentID. You then use that BatchID to publish, approve, or check the status of the content.

### 2. Content States

Content items in Agility CMS can be in one of these states:

| State Value | Name | Meaning |
|---|---|---|
| `0` | None | Not yet submitted |
| `1` | Published | Live on the website |
| `2` | Staging | Saved but not published |
| `3` | Deleted | Marked for deletion |
| `4` | Pending | Awaiting approval |

### 3. Workflow Operations

When publishing or approving content, you use these operation types:

| Value | Name | When to Use |
|---|---|---|
| `1` | Publish | Make content go live |
| `2` | Unpublish | Take content offline |
| `3` | Approve | Approve content in a workflow |
| `4` | Decline | Reject content in a workflow |
| `5` | RequestApproval | Ask someone to review and approve |

### 4. Locale

Most methods require a `locale` parameter — the language/region code of the content you're managing. Examples: `en-us`, `fr-ca`, `de-de`. Your Agility instance can support multiple locales.

### 5. GUID

The `guid` identifies your specific Agility CMS instance. Think of it as a unique ID for your "workspace". You'll use it in almost every method call. Find it in **Agility CMS → Settings → API Keys**.

### 6. Reference Names vs. Content IDs

- **ContentID** — A numeric ID (`22`) assigned to a specific content item. Use this when you know exactly which item you want.
- **ReferenceName** — A string identifier (`"blog-posts"`) that refers to a content *list* or *model*. Use this when you want to work with a category of content.

---

## Part 8: Method Classes Overview

The SDK organizes all operations into method groups, accessible through `ClientInstance`:

| Method Class | Property on ClientInstance | Operations | Methods Count |
|---|---|---|---|
| **ContentMethods** | `contentMethods` | CRUD for content items, publishing, workflow | ~13 |
| **AssetMethods** | `assetMethods` | Upload, move, delete files, manage galleries | ~15 |
| **PageMethods** | `pageManageMethods` | CRUD for pages, page trees, templates | ~17 |
| **ModelMethods** | `modelMethods` | CRUD for content model definitions | ~6 |
| **ContainerMethods** | `containerMethods` | Manage content containers and lists | ~8 |
| **BatchMethods** | `batchMethods` | Publish, approve, decline, check batches | ~7 |
| **InstanceMethods** | `instanceMethods` | Locales, instance-level info | ~1+ |

Think of these like departments in a company:
- `contentMethods` → Editorial department (writes and edits articles)
- `assetMethods` → Media/photography department (manages images and files)
- `pageManageMethods` → Web team (builds and organizes pages)
- `modelMethods` → Architecture team (designs the structure of content)
- `containerMethods` → Library team (organizes content into collections)
- `batchMethods` → Publishing/approvals team (decides what goes live and when)

---

## Part 9: ContentMethods — Managing Content Items

Access via: `clientInstance.contentMethods`

These are the most commonly used methods. They let you read, create, update, delete, and manage the publishing workflow for content items.

### Method Reference Table

| Method | Purpose | Key Parameters |
|---|---|---|
| `GetContentItem` | Fetch a single item by ID | `contentID`, `guid`, `locale` |
| `GetContentItems` | Fetch multiple items | `guid`, `locale`, `referenceName` |
| `GetContentList` | Fetch a paginated list | `guid`, `locale`, `referenceName` |
| `SaveContentItem` | Create or update an item | `contentItem`, `guid`, `locale` |
| `SaveContentItems` | Create/update multiple items | `contentItems[]`, `guid`, `locale` |
| `DeleteContent` | Delete a content item | `contentID`, `guid`, `locale` |
| `PublishContent` | Publish a single item | `contentID`, `guid`, `locale` |
| `UnPublishContent` | Unpublish a single item | `contentID`, `guid`, `locale` |
| `ApproveContent` | Approve in workflow | `contentID`, `guid`, `locale` |
| `DeclineContent` | Decline in workflow | `contentID`, `guid`, `locale` |
| `ContentRequestApproval` | Request approval | `contentID`, `guid`, `locale` |
| `GetContentHistory` | Get version history | `contentID`, `guid`, `locale` |
| `GetContentComments` | Get comments on item | `contentID`, `guid`, `locale` |

### Detailed Examples

#### **1. GetContentItem — Read a Single Item**

```csharp
// Get a content item by its numeric ID
var contentItem = await clientInstance.contentMethods.GetContentItem(
    22,       // contentID
    guid,     // your instance GUID
    locale    // "en-us"
);

Console.WriteLine(contentItem.fields.title);
```

**Use when:** You know the exact ID of the item you want to read before modifying it.

---

#### **2. GetContentList — Read a List of Items**

```csharp
// Get a list of content items from a container
var contentList = await clientInstance.contentMethods.GetContentList(
    guid,
    locale,
    "blog-posts",  // referenceName of the content list
    50,            // pageSize
    0              // rowIndex (for pagination)
);

foreach (var item in contentList.items)
{
    Console.WriteLine(item.fields.title);
}
```

**Use when:** You want to read all items in a content list (e.g. to update them in bulk).

---

#### **3. SaveContentItem — Create or Update an Item**

```csharp
// Create a NEW content item (contentID = 0 means new)
var newItem = new ContentItem
{
    contentID = 0,  // 0 = create new
    fields = new Dictionary<string, object>
    {
        { "title", "My New Blog Post" },
        { "body",  "This is the content body." },
        { "author", "Jane Doe" }
    },
    properties = new ContentItemProperties
    {
        referenceName = "blog-posts",  // which container to save to
        definitionName = "BlogPost",   // content model name
        state = 2                      // 2 = staging (not yet published)
    }
};

// SaveContentItem returns a BatchID, not a ContentID
int batchID = await clientInstance.contentMethods.SaveContentItem(
    newItem,
    guid,
    locale
);

Console.WriteLine($"Content saved in batch: {batchID}");
```

**To UPDATE an existing item**, pass its real contentID instead of 0:

```csharp
var existingItem = await clientInstance.contentMethods.GetContentItem(22, guid, locale);
existingItem.fields.title = "Updated Title";

int batchID = await clientInstance.contentMethods.SaveContentItem(
    existingItem,
    guid,
    locale
);
```

---

#### **4. SaveContentItems — Create Multiple Items at Once**

```csharp
// Create multiple items in a single batch — much more efficient than
// calling SaveContentItem in a loop
var items = new List<ContentItem>
{
    new ContentItem { fields = new { title = "Post 1" }, properties = new { referenceName = "blog-posts" } },
    new ContentItem { fields = new { title = "Post 2" }, properties = new { referenceName = "blog-posts" } },
    new ContentItem { fields = new { title = "Post 3" }, properties = new { referenceName = "blog-posts" } },
};

int batchID = await clientInstance.contentMethods.SaveContentItems(
    items,
    guid,
    locale
);

Console.WriteLine($"3 items saved in batch: {batchID}");
```

**Use for:** Bulk content imports (e.g. migrating hundreds of items from a CSV or legacy CMS).

---

#### **5. DeleteContent — Delete an Item**

```csharp
// Delete a content item permanently
await clientInstance.contentMethods.DeleteContent(
    22,     // contentID to delete
    guid,
    locale
);
```

⚠️ **Caution:** Deletion is permanent in most cases. Consider unpublishing first if you might need the content later.

---

#### **6. PublishContent — Publish a Single Item**

```csharp
// After saving, publish a single item directly (bypassing batches)
await clientInstance.contentMethods.PublishContent(
    22,     // contentID
    guid,
    locale
);
```

---

#### **7. Workflow Methods**

```csharp
// Request that someone reviews this item before it goes live
await clientInstance.contentMethods.ContentRequestApproval(22, guid, locale);

// Approve the item (if you are an approver)
await clientInstance.contentMethods.ApproveContent(22, guid, locale);

// Reject the item with a reason
await clientInstance.contentMethods.DeclineContent(22, guid, locale);

// Unpublish (take offline without deleting)
await clientInstance.contentMethods.UnPublishContent(22, guid, locale);
```

---

## Part 10: AssetMethods — Managing Files & Media

Access via: `clientInstance.assetMethods`

These methods let you upload images and files, organize them into galleries and folders, and retrieve or delete them.

### Method Reference Table

| Method | Purpose |
|---|---|
| `Upload` | Upload one or more files to Agility |
| `MoveFile` | Move a file to a different folder |
| `DeleteFile` | Delete a file permanently |
| `GetMediaList` | List all media files |
| `GetGalleries` | List all media galleries |
| `GetGalleryById` | Get a specific gallery by ID |
| `GetGalleryByName` | Get a specific gallery by name |
| `GetDefaultContainer` | Get the default media container |
| `SaveGallery` | Create or update a gallery |
| `DeleteGallery` | Delete a gallery |
| `GetAssetByID` | Get a specific asset by ID |
| `GetAssetByUrl` | Get a specific asset by its URL |
| `CreateFolder` | Create a new folder in the media library |
| `DeleteFolder` | Delete a folder |
| `RenameFolder` | Rename a folder |

### Detailed Examples

#### **1. Upload — Upload Files**

```csharp
// Upload files using a dictionary:
// Key = filename, Value = local folder path where the file lives
var filesToUpload = new Dictionary<string, string>
{
    { "hero-image.jpg", "/local/path/to/images/" },
    { "product-photo.png", "/local/path/to/images/" }
};

var uploadedMedia = await clientInstance.assetMethods.Upload(
    filesToUpload,          // files to upload
    "/website/images/",     // target folder path in Agility's media library
    guid
);

Console.WriteLine($"Uploaded: {uploadedMedia.url}");
```

---

#### **2. GetMediaList — List All Media**

```csharp
// Get a paginated list of all assets in your media library
var mediaList = await clientInstance.assetMethods.GetMediaList(
    50,     // pageSize
    0,      // rowIndex (offset)
    guid
);

// Returns an AssetMediaList object
foreach (var asset in mediaList.assetMedia)
{
    Console.WriteLine($"{asset.fileName} → {asset.url}");
}
```

---

#### **3. GetGalleries — List All Galleries**

```csharp
// Get all galleries with optional search term
var galleries = await clientInstance.assetMethods.GetGalleries(
    guid,
    "product",  // optional search term
    50,         // pageSize
    0           // rowIndex
);

// Returns an AssetGalleries object
foreach (var gallery in galleries.assetMediaGroupings)
{
    Console.WriteLine($"Gallery: {gallery.galleryName}");
}
```

---

#### **4. MoveFile — Reorganize Media**

```csharp
// Move a file to a different location in the media library
var movedMedia = await clientInstance.assetMethods.MoveFile(
    "/website/images/old-folder/hero.jpg",  // current path
    "/website/images/new-folder/",           // new folder location
    guid
);
```

---

#### **5. CreateFolder — Organize Your Media Library**

```csharp
// Create a new folder structure in the media library
await clientInstance.assetMethods.CreateFolder(
    "/website/images/2026/march/",  // folder path to create
    guid
);
```

---

## Part 11: PageMethods — Managing Pages

Access via: `clientInstance.pageManageMethods`

These methods let you read and modify your site's page structure — creating pages, moving them in the tree, updating their settings, and controlling who can see them.

### Method Reference Table

| Method | Purpose |
|---|---|
| `GetPage` | Get a single page by ID |
| `GetPageByPath` | Get a page by its URL path |
| `GetPageList` | Get all pages as a flat list |
| `GetPageTree` | Get pages as a hierarchical tree |
| `GetPageHistory` | Get the version history of a page |
| `GetPageComments` | Get comments on a page |
| `GetPageListByPageTemplateID` | Get pages using a specific template |
| `GetPageListByPage` | Get child pages of a specific page |
| `GetPageTemplateList` | Get all available page templates |
| `GetPageSecurity` | Get security settings for a page |
| `GetPageItemTemplateList` | Get content zone templates for a page |
| `GetPageContentZones` | Get the content zones of a page |
| `SavePage` | Create or update a page |
| `SavePageSecurity` | Update security settings for a page |
| `MovePageItem` | Move a page to a different location in the tree |
| `DeletePage` | Delete a page |

### Detailed Examples

#### **1. GetPage — Read a Page**

```csharp
var page = await clientInstance.pageManageMethods.GetPage(
    1,      // pageID
    guid,
    locale
);

Console.WriteLine($"Page: {page.name} at path {page.path}");
```

---

#### **2. GetPageTree — Read the Full Site Structure**

```csharp
// Get all pages as a hierarchical tree — great for understanding site structure
var pageTree = await clientInstance.pageManageMethods.GetPageTree(
    guid,
    locale
);

// Recursively print the tree
void PrintTree(dynamic node, int depth = 0)
{
    Console.WriteLine($"{new string(' ', depth * 2)}{node.title} ({node.path})");
    foreach (var child in node.children)
        PrintTree(child, depth + 1);
}

foreach (var rootPage in pageTree)
    PrintTree(rootPage);
```

---

#### **3. SavePage — Create or Update a Page**

```csharp
// Create a new page
var newPage = new Page
{
    pageID = 0,              // 0 = create new
    name = "about-us",
    title = "About Us",
    menuText = "About",
    pageType = "static",
    templateName = "Default Template",
    parentPageID = -1        // -1 = root level page
};

int batchID = await clientInstance.pageManageMethods.SavePage(
    newPage,
    guid,
    locale
);

Console.WriteLine($"Page saved in batch: {batchID}");
```

---

#### **4. MovePageItem — Reorder or Reparent a Page**

```csharp
// Move a page to a different location in the site tree
await clientInstance.pageManageMethods.MovePageItem(
    5,      // pageID to move
    2,      // new parent pageID (or -1 for root)
    1,      // new sort order position
    guid,
    locale
);
```

---

#### **5. DeletePage — Remove a Page**

```csharp
// Delete a page and all its child pages
await clientInstance.pageManageMethods.DeletePage(
    5,      // pageID
    guid,
    locale
);
```

⚠️ **Caution:** Deleting a parent page also deletes all child pages. Check `GetPageTree` first to understand the impact.

---

## Part 12: ModelMethods — Managing Content Models

Access via: `clientInstance.modelMethods`

Content models define the **structure** (fields and field types) of your content. Think of a content model as the blueprint that determines what fields a Blog Post, Product, or Team Member has.

### Method Reference Table

| Method | Purpose |
|---|---|
| `GetContentModel` | Get a model definition by ID |
| `GetModelByReferenceName` | Get a model by its reference name |
| `GetContentModules` | List all content modules |
| `GetPageModules` | List all page modules |
| `SaveModel` | Create or update a content model |
| `DeleteModel` | Delete a content model |

### Detailed Examples

#### **1. GetModelByReferenceName — Read a Content Model**

```csharp
// Get the model definition for "BlogPost"
var model = await clientInstance.modelMethods.GetModelByReferenceName(
    "BlogPost",  // the reference name of the model
    guid
);

// Inspect the fields defined in this model
foreach (var field in model.fields)
{
    Console.WriteLine($"Field: {field.name} ({field.type})");
}
```

**Use when:** You want to understand what fields are available before saving content items of that type.

---

#### **2. SaveModel — Create a New Content Model**

```csharp
// Create a new content model (defines structure of a new content type)
var newModel = new ContentModel
{
    id = 0,
    referenceName = "ProductReview",
    displayName = "Product Review",
    fields = new List<ModelField>
    {
        new ModelField { name = "reviewerName", label = "Reviewer Name", type = "Text" },
        new ModelField { name = "rating",       label = "Rating",        type = "Number" },
        new ModelField { name = "body",         label = "Review Body",   type = "HTML" }
    }
};

await clientInstance.modelMethods.SaveModel(newModel, guid);
```

---

## Part 13: ContainerMethods — Managing Containers

Access via: `clientInstance.containerMethods`

A **Container** in Agility CMS is the list or collection that holds content items of a given type. Think of it as a database table — the model defines the columns, and the container stores the rows.

### Method Reference Table

| Method | Purpose |
|---|---|
| `GetContainerByID` | Get a container by numeric ID |
| `GetContainerByReferenceName` | Get a container by reference name |
| `GetContainersByModel` | Get all containers using a specific model |
| `GetContainerList` | List all containers in the instance |
| `GetContainerSecurity` | Get security settings for a container |
| `GetNotificationList` | Get notification settings |
| `SaveContainer` | Create or update a container |
| `DeleteContainer` | Delete a container |

### Detailed Examples

#### **1. GetContainerList — See All Containers**

```csharp
// List all content containers in your Agility instance
var containers = await clientInstance.containerMethods.GetContainerList(guid);

foreach (var container in containers)
{
    Console.WriteLine($"Container: {container.referenceName} ({container.contentDefinitionName})");
}
```

---

#### **2. GetContainerByReferenceName — Get a Specific Container**

```csharp
// Get full details of a specific container
var container = await clientInstance.containerMethods.GetContainerByReferenceName(
    "blog-posts",   // reference name
    guid
);

Console.WriteLine($"Container ID: {container.contentViewID}");
Console.WriteLine($"Model: {container.contentDefinitionName}");
```

---

#### **3. SaveContainer — Create a New Content List**

```csharp
// Create a new content container (list)
var newContainer = new Container
{
    contentViewID = 0,              // 0 = create new
    referenceName = "team-members",
    contentDefinitionName = "TeamMember",  // must match an existing model
    title = "Team Members"
};

await clientInstance.containerMethods.SaveContainer(newContainer, guid);
```

---

## Part 14: BatchMethods — Workflow & Publishing

Access via: `clientInstance.batchMethods`

Batches are at the heart of Agility's publishing workflow. Every time you save, create, or delete content, it creates a batch. You then use these methods to control what happens to that batch.

### Method Reference Table

| Method | Purpose |
|---|---|
| `GetBatch` | Get details and status of a batch |
| `GetBatchTypes` | Get all enum values for batch operations |
| `PublishBatch` | Publish all items in a batch |
| `UnpublishBatch` | Unpublish all items in a batch |
| `ApproveBatch` | Approve all items in a batch |
| `DeclineBatch` | Decline all items in a batch |
| `RequestApprovalBatch` | Request approval for a batch |

### Batch States

| State | Value | Meaning |
|---|---|---|
| None | 0 | Batch created, not submitted |
| Pending | 1 | Awaiting processing |
| InProcess | 2 | Currently being processed |
| Processed | 3 | Successfully completed |
| Deleted | 4 | Batch was deleted |

### Detailed Examples

#### **1. Complete Save → Publish Workflow**

This is the most important pattern to understand:

```csharp
// STEP 1: Save content — returns a BatchID
int batchID = await clientInstance.contentMethods.SaveContentItem(
    newItem, guid, locale
);
Console.WriteLine($"Saved. BatchID = {batchID}");

// STEP 2: Check the batch status
var batch = await clientInstance.batchMethods.GetBatch(batchID, guid);
Console.WriteLine($"Batch status: {batch.status}");

// STEP 3: Publish the batch to make it live
await clientInstance.batchMethods.PublishBatch(batchID, guid);
Console.WriteLine("Published!");
```

---

#### **2. Approval Workflow**

When your Agility instance uses content approval workflows:

```csharp
// STEP 1: Save content
int batchID = await clientInstance.contentMethods.SaveContentItem(draft, guid, locale);

// STEP 2: Request approval from an editor
await clientInstance.batchMethods.RequestApprovalBatch(batchID, guid);
Console.WriteLine("Sent for approval.");

// ─── (Editor reviews the content in Agility CMS) ───

// STEP 3a: Editor approves
await clientInstance.batchMethods.ApproveBatch(batchID, guid);

// OR

// STEP 3b: Editor declines
await clientInstance.batchMethods.DeclineBatch(batchID, guid);
```

---

#### **3. GetBatchTypes — Discover Available Enums**

```csharp
// Useful during development to see all valid operation types
var batchTypes = await clientInstance.batchMethods.GetBatchTypes(guid);

Console.WriteLine("Workflow Operations:");
foreach (var op in batchTypes.workflowOperations)
{
    Console.WriteLine($"  {op.value}: {op.name} — {op.description}");
}

Console.WriteLine("\nBatch States:");
foreach (var state in batchTypes.states)
{
    Console.WriteLine($"  {state.value}: {state.name}");
}
```

---

## Part 15: InstanceMethods & User Methods

### InstanceMethods — Access via `clientInstance.instanceMethods`

```csharp
// Get all locales configured for your Agility instance
var locales = await clientInstance.instanceMethods.GetLocales(guid);

foreach (var locale in locales)
{
    Console.WriteLine($"Locale: {locale.languageCode} — {locale.name}");
}
```

**Use when:** Building tools that need to manage content across multiple languages.

### User Management (InstanceUserMethods)

If your version of the SDK includes user management:

```csharp
// Get all users for your instance
var users = await clientInstance.instanceMethods.GetUsers(guid);

// Add a new user
await clientInstance.instanceMethods.SaveUser(
    new User { email = "newuser@company.com", roleName = "Editor" },
    guid
);

// Remove a user
await clientInstance.instanceMethods.DeleteUser("user@company.com", guid);
```

---

## Part 16: Common Patterns You'll See

### Pattern 1: Always Await — Everything is Async

All SDK methods return `Task<T>`, meaning they are asynchronous. Always use `await`:

```csharp
// CORRECT
var item = await clientInstance.contentMethods.GetContentItem(22, guid, locale);

// INCORRECT — will cause deadlocks or unexpected behavior
var item = clientInstance.contentMethods.GetContentItem(22, guid, locale).Result;
```

### Pattern 2: Save Returns BatchID, Not ContentID

A common beginner mistake:

```csharp
// SaveContentItem does NOT return the new contentID
// It returns a BatchID
int batchID = await clientInstance.contentMethods.SaveContentItem(item, guid, locale);

// To get the new ContentID, you need to query the content list after saving
```

### Pattern 3: ContentID = 0 Means Create New

```csharp
// contentID = 0 → CREATE a new item
new ContentItem { contentID = 0, ... }

// contentID = 22 → UPDATE the existing item with ID 22
new ContentItem { contentID = 22, ... }
```

### Pattern 4: Always Wrap in Try/Catch

```csharp
try
{
    var item = await clientInstance.contentMethods.GetContentItem(22, guid, locale);
    if (item == null)
    {
        Console.WriteLine("Item not found.");
        return;
    }
    // use item
}
catch (Exception ex)
{
    Console.Error.WriteLine($"API error: {ex.Message}");
}
```

### Pattern 5: Guid and Locale Appear in Almost Every Method

```csharp
// Rather than repeating these everywhere, store them as variables:
string guid   = Environment.GetEnvironmentVariable("AGILITY_GUID");
string locale = "en-us";

// Then reuse:
await clientInstance.contentMethods.GetContentItem(22, guid, locale);
await clientInstance.contentMethods.GetContentList(guid, locale, "blog-posts", 50, 0);
await clientInstance.pageManageMethods.GetPage(1, guid, locale);
```

### Pattern 6: Bulk Operations are More Efficient

```csharp
// SLOW — 100 separate API calls
foreach (var item in itemsToCreate)
{
    await clientInstance.contentMethods.SaveContentItem(item, guid, locale);
}

// FAST — 1 API call for all 100 items
await clientInstance.contentMethods.SaveContentItems(itemsToCreate, guid, locale);
```

---

## Part 17: Error Handling

The Management SDK throws exceptions when API calls fail. Unlike the Fetch SDK (which returns `null` on 404), this SDK uses standard .NET exception handling.

### Common Error Scenarios

```csharp
try
{
    var item = await clientInstance.contentMethods.GetContentItem(999, guid, locale);
}
catch (HttpRequestException ex) when (ex.Message.Contains("404"))
{
    Console.WriteLine("Content item not found.");
}
catch (HttpRequestException ex) when (ex.Message.Contains("401"))
{
    Console.WriteLine("Authentication failed. Check your Bearer token.");
}
catch (HttpRequestException ex) when (ex.Message.Contains("403"))
{
    Console.WriteLine("Forbidden. Your token may not have permission for this operation.");
}
catch (Exception ex)
{
    Console.Error.WriteLine($"Unexpected error: {ex.Message}");
}
```

### Token Expiry

Bearer tokens expire. If you get a `401 Unauthorized` error, it likely means your token has expired. Refresh it using the OAuth refresh endpoint (see Part 6).

```csharp
// Pattern for handling token expiry with retry
async Task<T> WithTokenRefresh<T>(Func<Task<T>> apiCall)
{
    try
    {
        return await apiCall();
    }
    catch (HttpRequestException ex) when (ex.Message.Contains("401"))
    {
        // Refresh the token
        options.token = await RefreshToken(refreshToken);
        clientInstance = new ClientInstance(options);
        
        // Retry the call
        return await apiCall();
    }
}
```

---

## Part 18: Learning Plan Step by Step

### Phase 1: Foundation (Days 1–2)
**Goal:** Understand what the Management SDK does and how it differs from the Fetch SDK.

- Read Part 1 and Part 2 of this guide
- Sign up for a free Agility CMS trial at https://agilitycms.com/free
- Log in to the Agility CMS interface and explore content, pages, and media manually
- Read the official README in the repository

**Self-check questions:**
- Can you explain the difference between the Fetch SDK and Management SDK?
- Can you explain what a batch is and why it exists?

---

### Phase 2: Setup & First Request (Days 3–4)
**Goal:** Get the SDK installed and make your first API call.

- Clone the repository locally
- Set up your credentials (token, guid, locale) using environment variables
- Create a simple console app and call `GetContentItem()` to verify your setup
- Explore the response object in your debugger

```csharp
// Your first program — verify credentials work
var item = await clientInstance.contentMethods.GetContentItem(
    22, // use a real contentID from your Agility instance
    guid,
    locale
);
Console.WriteLine(item?.fields?.title ?? "Item not found");
```

**Self-check questions:**
- Did you get a successful response?
- Can you read the `fields` properties of the returned content item?

---

### Phase 3: Read Methods (Days 5–6)
**Goal:** Explore all read (`Get...`) methods across all method classes.

Practice these in order:
1. `contentMethods.GetContentList()` — list all items in a container
2. `pageManageMethods.GetPageTree()` — understand your site structure
3. `assetMethods.GetMediaList()` — see all media in your library
4. `modelMethods.GetContentModel()` — inspect a content model's fields
5. `containerMethods.GetContainerList()` — see all containers

**Self-check questions:**
- What fields does your "Blog Post" model have?
- How many pages are in your site tree?

---

### Phase 4: Write Methods (Days 7–9)
**Goal:** Create, update, and delete content.

Practice in this order (start with less risky operations):
1. Create a new content item with `SaveContentItem()`
2. Check the returned BatchID with `GetBatch()`
3. Publish it with `PublishBatch()`
4. Update the same item (pass its real contentID)
5. Unpublish it with `UnpublishBatch()`
6. Delete it with `DeleteContent()`

**Self-check questions:**
- What is the difference between `SaveContentItem()` and `PublishBatch()`?
- Why does `SaveContentItem()` return a BatchID?

---

### Phase 5: Asset & Page Management (Days 10–11)
**Goal:** Learn media and page operations.

1. Upload a test image with `assetMethods.Upload()`
2. Create a test folder with `assetMethods.CreateFolder()`
3. Move the image into the new folder with `assetMethods.MoveFile()`
4. Create a test page with `pageManageMethods.SavePage()`
5. Move it in the tree with `pageManageMethods.MovePageItem()`
6. Delete your test page and image

---

### Phase 6: Real Project (Days 12–14)
**Goal:** Build something practical.

Ideas:
- **Content importer** — Read a CSV file and create content items from each row
- **Bulk publisher** — Get all items in a container with state=2 (staging) and publish them all
- **Media organizer** — Get all assets and move them into organized folders by type or date
- **Content audit tool** — List all pages and their associated content, generate a report

**Milestones:**
- ✅ SDK installed and authenticated
- ✅ Can read content, pages, and assets
- ✅ Can create and update content items
- ✅ Understand the batch/publish workflow
- ✅ Handled at least one error case gracefully
- ✅ Built a real utility or script

---

## Part 19: Key Files to Study

### Priority 1 — MUST READ (2–3 hours total)

| File | Time | Key Concept |
|---|---|---|
| `README.md` in the repo | 10 min | Overview and quick start |
| `Models/Options.cs` | 5 min | Configuration — token, locale, guid |
| `ClientInstance.cs` | 10 min | How method groups are exposed |
| `Methods/ContentMethods.cs` | 25 min | Most important method class |
| `Methods/BatchMethods.cs` | 15 min | Publishing and workflow |

### Priority 2 — IMPORTANT (2–3 hours total)

| File | Time | Key Concept |
|---|---|---|
| `Methods/AssetMethods.cs` | 20 min | File upload and media management |
| `Methods/PageManageMethods.cs` | 20 min | Page CRUD and tree navigation |
| `Models/ContentItem.cs` | 10 min | Shape of a content item |
| `Models/Media.cs` | 10 min | Shape of an asset |
| `Models/AssetMediaList.cs` | 5 min | Paginated media response |

### Priority 3 — REFERENCE

| File | When Needed |
|---|---|
| `Methods/ModelMethods.cs` | When managing content model definitions |
| `Methods/ContainerMethods.cs` | When managing content lists/containers |
| `Methods/InstanceMethods.cs` | When working with locales or users |
| `management.api.sdk.test/` | When contributing or debugging |

---

## Part 20: Questions to Ask Yourself While Learning

### About the Architecture
- Why does `ClientInstance` expose multiple method groups instead of one big class?
- What does `ClientInstance` store internally? (Answer: the `Options` config)
- Why does the SDK need a Bearer token when the Fetch SDK only needs an API key?

### About Batches
- What happens to content after `SaveContentItem()` before `PublishBatch()` is called?
- Why would you use `RequestApprovalBatch()` instead of `PublishBatch()` directly?
- Can you publish part of a batch, or is it all-or-nothing?
- What is the difference between batch state `Pending` and `InProcess`?

### About Content
- What does `contentID = 0` mean when passed to `SaveContentItem()`?
- What is the difference between `SaveContentItem()` and `SaveContentItems()`?
- Why does deleting a content item sometimes require unpublishing it first?
- What is a `referenceName` and how does it differ from a `contentID`?

### About Models & Containers
- What is the difference between a content model and a container?
- Can two containers share the same content model?
- What happens if you try to save a content item to a container it doesn't belong to?

### About Error Handling
- What HTTP status code do you get when your token expires?
- What is the difference between a 401 and a 403 error?
- How would you build a retry mechanism for expired tokens?

---

## Part 21: Quick Reference

### Setup in 4 Lines

```csharp
using management.api.sdk;
using agility.models;

Options options = new Options { token = "YOUR_TOKEN", locale = "en-us", guid = "YOUR_GUID" };
ClientInstance client = new ClientInstance(options);
```

### Most Common Operations

```csharp
// READ
var item     = await client.contentMethods.GetContentItem(22, guid, locale);
var list     = await client.contentMethods.GetContentList(guid, locale, "blog-posts", 50, 0);
var page     = await client.pageManageMethods.GetPage(1, guid, locale);
var tree     = await client.pageManageMethods.GetPageTree(guid, locale);
var media    = await client.assetMethods.GetMediaList(50, 0, guid);

// WRITE
int batchID  = await client.contentMethods.SaveContentItem(item, guid, locale);
int batchID2 = await client.contentMethods.SaveContentItems(items, guid, locale);
int batchID3 = await client.pageManageMethods.SavePage(page, guid, locale);

// PUBLISH
await client.batchMethods.PublishBatch(batchID, guid);

// DELETE
await client.contentMethods.DeleteContent(22, guid, locale);
await client.pageManageMethods.DeletePage(5, guid, locale);
```

### The Save → Publish Pattern

```csharp
// STEP 1: Save → get batch
int batchID = await client.contentMethods.SaveContentItem(myItem, guid, locale);

// STEP 2: Verify batch (optional)
var batch = await client.batchMethods.GetBatch(batchID, guid);

// STEP 3: Publish
await client.batchMethods.PublishBatch(batchID, guid);
```

---

## Part 22: Contributing to the SDK

The SDK is open source. If you find a bug or want to add a feature:

1. **Fork** the repository at https://github.com/agility/agility-cms-management-sdk-dotnet
2. **Clone** your fork:
   ```bash
   git clone https://github.com/YOUR-USERNAME/agility-cms-management-sdk-dotnet.git
   ```
3. **Open** the solution in Visual Studio or JetBrains Rider
4. **Make your changes** in the `management.api.sdk` project
5. **Add or update tests** in `management.api.sdk.test`
6. **Run the tests** to verify nothing is broken:
   ```bash
   dotnet test
   ```
7. **Submit a Pull Request** with a clear description of your change

### Running the Tests

The test project (`management.api.sdk.test`) requires real Agility CMS credentials. You'll need to supply your token, guid, and locale in a `.runsettings` file before running:

```xml
<!-- .runsettings -->
<RunSettings>
  <TestRunParameters>
    <Parameter name="token" value="YOUR_BEARER_TOKEN" />
    <Parameter name="guid"  value="YOUR_GUID" />
    <Parameter name="locale" value="en-us" />
  </TestRunParameters>
</RunSettings>
```

Then run:
```bash
dotnet test --settings .runsettings
```

### Reporting Issues

Open an issue at https://github.com/agility/agility-cms-management-sdk-dotnet/issues

Include: your .NET version, SDK version, a minimal code reproduction, and the error message/stack trace.

---

## Part 23: Official Resources & Links

| Resource | URL |
|---|---|
| GitHub Repository | https://github.com/agility/agility-cms-management-sdk-dotnet |
| TypeScript equivalent (same API surface) | https://github.com/agility/agility-cms-management-sdk-typescript |
| Agility CMS Help Center | https://help.agilitycms.com/hc/en-us |
| Agility CMS Docs | https://agilitycms.com/docs |
| .NET Docs (ASP.NET) | https://agilitycms.com/docs/dotNet |
| Free Trial | https://agilitycms.com/free |
| Community Slack | https://join.slack.com/t/agilitycommunity/shared_invite/... |
| Developer Community Forum | https://help.agilitycms.com/hc/en-us/community/topics |

---

## Part 24: Glossary of Terms

**Async/Await** — C# keywords for writing asynchronous code. All SDK methods use this pattern.

**Batch** — A collection of content changes grouped together for publishing or workflow processing.

**BatchID** — The numeric ID returned when you save content. Use it to publish or check status.

**Bearer Token** — A security credential (JWT) passed in the `Authorization` HTTP header. Required for all Management SDK calls.

**ClientInstance** — The main entry point of the .NET Management SDK. Contains all method group properties.

**Container** — A content list in Agility CMS. Holds multiple content items of the same model type.

**Content Model** — A definition of the fields (schema) for a type of content. Like a class definition in C#.

**ContentID** — A unique numeric identifier for a specific content item.

**CRUD** — Create, Read, Update, Delete — the four basic operations on data.

**GUID** — A unique identifier for your Agility CMS instance. Used in almost every API call.

**Locale** — Language and region code for content (e.g. `en-us`, `fr-ca`).

**Management API** — Agility's REST API for writing and managing content. The SDK wraps this API.

**NuGet** — The package manager for .NET, equivalent to npm for JavaScript.

**OAuth 2.0** — An authentication standard used to obtain Bearer tokens securely.

**Options** — The C# class that holds your SDK configuration: token, locale, guid.

**ReferenceName** — A string identifier for a content model or container (e.g. `"blog-posts"`).

**State** — The status of a content item: None, Published, Staging, Deleted, or Pending.

**Task\<T\>** — The C# return type for async methods. Always use `await` when calling SDK methods.

**Workflow** — A process for reviewing and approving content before it goes live (request → approve/decline → publish).

---

## Part 25: FAQ

**Q: What is the difference between the Management SDK and the Fetch SDK?**
A: The Fetch SDK is read-only (gets published content for your website). The Management SDK can create, update, delete, and publish content — it is for admin/automation tasks.

**Q: Do I need to know C# to use this SDK?**
A: Yes. This SDK is written in C# and targets .NET developers. If you prefer JavaScript/TypeScript, Agility also has `agility-cms-management-sdk-typescript` with the same capabilities.

**Q: Why does `SaveContentItem()` return a BatchID instead of a ContentID?**
A: Because in Agility CMS, saved content is not published immediately. It enters a batch (a queue) first. The BatchID lets you track that batch, publish it, or route it through an approval workflow.

**Q: How do I find my GUID and generate a Bearer token?**
A: Log in to `app.agilitycms.com` → Settings → API Keys. Your GUID is listed there, and you can generate a Management API token.

**Q: Can I use this SDK in a public-facing website?**
A: No. Bearer tokens must be kept secret. Only use this SDK in server-side code, CLI tools, CI/CD pipelines, or internal admin applications — never in client-side JavaScript or mobile apps.

**Q: What happens if I save content but never call `PublishBatch()`?**
A: The content exists in Agility CMS in "Staging" state (state = 2). It will NOT be visible on your live website via the Fetch SDK. It stays in the batch queue until published, declined, or deleted.

**Q: Can I delete a batch after creating it?**
A: Yes. A batch in state `None` or `Pending` can typically be deleted. Check the `BatchMethods` class for a `DeleteBatch` method in your version of the SDK.

**Q: Is there a way to publish content without going through a batch?**
A: Yes. `contentMethods.PublishContent()` lets you publish a single item directly. However, for multiple items or workflow-controlled instances, using batches is recommended.

**Q: What .NET version do I need?**
A: .NET 6.0 or later is recommended. Check the `management.api.sdk.csproj` file in the repo for the exact target framework.

**Q: Is there a sandbox/test instance I can use while learning?**
A: Yes. Sign up for a free Agility CMS account at https://agilitycms.com/free. This gives you a full instance to experiment with without affecting any production content.

---

**Last Updated:** 2026-03-04
**Guide Version:** 1.0
**Created by:** Claude (Anthropic)

**Happy Learning! 🚀**
