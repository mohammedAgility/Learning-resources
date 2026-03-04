# 📚 Learning Resources — Agility CMS

A personal collection of beginner-friendly learning guides for working with **Agility CMS** and its JavaScript SDKs.

---

## 📂 Contents

| File | Description |
|---|---|
| [`AGILITY_FETCH_SDK_COMPLETE_BEGINNER_GUIDE.md`](./AGILITY_FETCH_SDK_COMPLETE_BEGINNER_GUIDE.md) | Complete beginner's guide to the Agility Content Fetch JS SDK — architecture, methods, types, learning plan, and more |
| [`aility-cms-dotnet-mgmt-sdk-guide.md`](./aility-cms-dotnet-mgmt-sdk-guide.md) | Complete beginner's guide to the Agility CMS Management SDK for .NET — authentication, content/asset/page/model/container/batch management, learning plan, and more |

---

## 🚀 About This Repo

This repository contains in-depth learning materials designed for **complete beginners** who want to understand how to build with [Agility CMS](https://agilitycms.com). The guides go far beyond the official documentation by covering:

- Internal architecture and code structure
- Step-by-step data flow explanations
- All available SDK methods with full examples
- TypeScript types and how to use generics
- Structured learning plans with self-check questions
- Real-world patterns and best practices

---

## 📖 Guides

### 🎓 Complete Beginner's Guide to the Agility Content Fetch JS SDK

**File:** [`AGILITY_FETCH_SDK_COMPLETE_BEGINNER_GUIDE.md`](./AGILITY_FETCH_SDK_COMPLETE_BEGINNER_GUIDE.md)

**Estimated learning time:** 10–14 days (1–2 hours/day)
**Difficulty:** Beginner

A comprehensive guide to the [`@agility/content-fetch`](https://www.npmjs.com/package/@agility/content-fetch) JavaScript SDK — the official library for reading live and preview content from an Agility CMS instance.

**What's covered:**

- What the SDK is and why it exists
- Full repository structure breakdown (`src/`, `methods/`, `types/`)
- How data flows end-to-end from your app to Agility's servers and back
- Core concepts: configuration, live vs. preview mode, caching, error handling
- All 10 SDK methods with parameters, examples, and return shapes
- TypeScript generics and how to type your own content fields
- Installation via npm, yarn, and CDN script tag
- A 5-phase, day-by-day learning plan
- Common code patterns used across the SDK
- Contributing to the SDK and running tests
- Content webhooks for real-time updates
- Official tutorials and resources

---

### 🎓 Complete Beginner's Guide to the Agility CMS Management SDK for .NET

**File:** [`aility-cms-dotnet-mgmt-sdk-guide.md`](./aility-cms-dotnet-mgmt-sdk-guide.md)

**Estimated learning time:** 10–14 days (1–2 hours/day)
**Difficulty:** Beginner

A comprehensive guide to the Agility CMS Management SDK for .NET — the official C# library for creating, updating, deleting, and managing content, pages, assets, models, and users in Agility CMS programmatically.

**What's covered:**

- What the SDK is, why it exists, and how it differs from the Fetch SDK
- Full repository structure breakdown (`Methods/`, `Models/`, `ClientInstance.cs`)
- End-to-end data flow from your .NET app to the Agility Management API and back
- OAuth 2.0 Bearer Token authentication and configuration
- Core concepts: the Batch system, content states, workflow operations, locales, GUIDs
- All method classes with full examples: `ContentMethods`, `AssetMethods`, `PageMethods`, `ModelMethods`, `ContainerMethods`, `BatchMethods`, `InstanceMethods`
- The Save → Publish batch workflow pattern
- Installation via NuGet or project reference
- A 6-phase, day-by-day learning plan
- Common patterns and best practices (async/await, error handling, bulk operations)
- Contributing to the SDK and running tests
- Glossary of terms and FAQ

---

## 🛠️ Related Links

| Resource | URL |
|---|---|
| Agility Content Fetch JS SDK (GitHub) | https://github.com/agility/agility-content-fetch-js-sdk |
| npm Package | https://www.npmjs.com/package/@agility/content-fetch |
| Agility CMS Management SDK for .NET (GitHub) | https://github.com/agility/agility-cms-management-sdk-dotnet |
| Agility CMS Management SDK for TypeScript (GitHub) | https://github.com/agility/agility-cms-management-sdk-typescript |
| Agility CMS | https://agilitycms.com |
| Agility Help Center | https://help.agilitycms.com/hc/en-us |
| Free Trial | https://agilitycms.com/free |

---

## 🤝 Contributing

This is a personal learning repository, but suggestions and corrections are welcome. If you spot an inaccuracy or want to add a guide, feel free to open an issue or submit a pull request.

---

## 📄 License

This repository is for educational purposes. All code examples are based on the [Agility Content Fetch JS SDK](https://github.com/agility/agility-content-fetch-js-sdk), which is licensed under the MIT License.
