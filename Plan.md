# Gym Website + Admin CMS — Complete Plan

## Architecture

```text
┌─────────────────────────────────────┐
│         React + Vite Frontend        │
├──────────────┬──────────────────────┤
│  Public Site │   Admin Dashboard    │
│  (Gym pages) │   (/admin/*)         │
├──────────────┴──────────────────────┤
│           Firebase Backend           │
│  Auth | Firestore | Storage         │
└─────────────────────────────────────┘
```

## Tech Stack
- React 18 + Vite + TypeScript
- Tailwind CSS + shadcn/ui (already installed)
- Firebase Auth, Firestore, Storage
- TipTap (rich text editor)
- react-helmet-async (SEO meta tags)
- React Query (caching), react-router-dom (routing)

## Project Structure

```text
src/
├── contexts/         AuthContext
├── layouts/          PublicLayout, AdminLayout
├── lib/              firebase.ts, utils.ts
├── types/            All TypeScript interfaces
├── hooks/            useAuth, useFirestore, useStorage
├── pages/
│   ├── public/       Home, About, Classes, Trainers, Blog, Contact, DynamicPage
│   └── admin/        Dashboard, Pages, Blogs, Navigation, Projects,
│                     SEO, Settings, SocialLinks, Files, Enquiries
├── components/
│   ├── public/       Navbar, Footer, Hero, ClassCard, TrainerCard, ContactForm
│   └── admin/        Sidebar, DataTable, RichTextEditor, FileUploader, SEOScoreCard
```

## Firestore Collections

| Collection | Key Fields |
|---|---|
| `pages` | title, slug, content, seoTitle, seoDescription, seoKeywords, ogImage, canonicalUrl, status (draft/published), order, createdAt, updatedAt |
| `blogs` | title, slug, content, excerpt, featuredImage, categoryId, tags[], status, publishedAt, createdAt |
| `blog_categories` | name, slug |
| `navigation_items` | label, url, parentId, position (header/footer), order, target (_blank/_self) |
| `projects` | title, description, images[], category, order, createdAt |
| `business_settings` | (single doc) name, address, phone, email, hours{}, logo, favicon, mapEmbed, geoCoordinates |
| `social_links` | platform, url, icon, order |
| `enquiries` | name, email, phone, subject, message, type (contact/class/membership), status (new/read/replied/archived), starred, createdAt, repliedAt |
| `seo_settings` | (single doc) siteTitle, titleTemplate, defaultDescription, defaultOgImage, robotsTxt, analyticsId, tagManagerId, schemaOrgData |
| `redirects` | fromPath, toPath, type (301/302) |
| `user_roles` | userId, role (admin) |
| `files` | name, url, size, type, folder, uploadedAt |

## Phase 1 — Foundation
1. Install Firebase SDK, create `src/lib/firebase.ts` config
2. AuthContext with login/logout, admin role check via `user_roles` collection
3. ProtectedRoute wrapper for `/admin/*`
4. PublicLayout (dark/bold gym navbar + footer) and AdminLayout (sidebar + topbar)
5. All route definitions in App.tsx

## Phase 2 — Admin Dashboard Shell
1. Admin sidebar with all menu items: Dashboard, Pages, Blogs, Navigation, Projects, Enquiries, SEO, Settings, Social Links, Files
2. Dashboard home with stat cards (page count, blog count, unread enquiries, etc.)
3. Reusable DataTable component for all CRUD lists
4. Reusable RichTextEditor component (TipTap)

## Phase 3 — Admin CRUD Modules
1. **Pages** — List, create, edit, delete. Rich text content, SEO fields per page, slug auto-generation, draft/published toggle, ordering
2. **Blogs** — List, create, edit, delete. Categories & tags, featured image upload, excerpt, scheduled publishing
3. **Navigation** — Visual menu builder, drag-to-reorder, nested items (parent/child), header vs footer menus
4. **Projects/Portfolio** — CRUD with multi-image upload, categories (Transformations, Facility, Events)
5. **Business Settings** — Single form: gym name, address, phone, email, business hours editor, logo/favicon upload, map embed
6. **Social Links** — CRUD with platform picker, URL, icon, drag-to-reorder
7. **File Manager** — Upload/browse/delete via Firebase Storage, folder organization, image preview, select files for use in pages/blogs

## Phase 4 — Enquiries Management
1. **Public forms**: Contact form + Class enquiry form (Zod validated, stored in Firestore)
2. **Admin `/admin/enquiries`**: Data table with filters (date, status, type), status workflow (New → Read → Replied → Archived), detail view, mark starred, delete, export CSV
3. **Sidebar badge**: Unread enquiry count on sidebar nav item

## Phase 5 — Comprehensive SEO
1. **Admin `/admin/seo`**: Global SEO defaults (title template, default description, default OG image), robots.txt editor, sitemap settings, Google Analytics/Tag Manager ID, Schema.org LocalBusiness settings, redirect manager (old → new path)
2. **Per-page SEO**: Already part of Pages/Blogs CRUD — meta title, description, keywords, OG image, canonical URL, SEO score indicator (title length, description length, heading check)
3. **Runtime SEO (react-helmet-async)**: Dynamic `<title>`, `<meta>` description/keywords, Open Graph tags (og:title, og:description, og:image, og:url), Twitter Card tags, canonical `<link>`, JSON-LD structured data (LocalBusiness for gym, BlogPosting for blog posts, BreadcrumbList)
4. **Auto-generated routes**: `/sitemap.xml` from published pages/blogs, `/robots.txt` from admin settings
5. **Redirect handling**: Middleware-style check on page load for 301 redirects

## Phase 6 — Public Gym Website (Dark & Bold Theme)
1. **Home**: Hero section with CTA (Join Now), featured classes grid, trainer highlights, testimonials, pricing preview, contact/location section — all pulling from Firestore
2. **Classes**: Grid with filter by type (Yoga, CrossFit, Boxing, etc.)
3. **Trainers**: Cards with photo, bio, specialties
4. **Blog**: Grid listing with pagination + detail page (`/blog/:slug`) with full SEO meta
5. **Contact**: Contact form + class enquiry form, Google Maps embed, business hours from settings
6. **Dynamic Pages**: Render any admin-created page by slug (`/:slug`)
7. **Navigation**: Header/footer menus rendered from Firestore navigation_items
8. **Footer**: Social links, business info, quick nav links — all from Firestore

## Firebase Security Rules
- Public read on published pages, blogs, navigation, projects, business_settings, social_links
- Public write on enquiries (rate-limited)
- Admin-only read/write on all collections for CRUD operations
- Admin role verified via `user_roles` collection match on `request.auth.uid`

## Implementation Order
Phase 1 → Phase 2 → Phase 3 → Phase 4 → Phase 5 → Phase 6

Each phase builds on the previous. The admin CMS is built first so content exists when we build the public site.