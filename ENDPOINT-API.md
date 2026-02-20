# API Documentation

Base URL: `deramaserver.vercel.app/api`

This document provides details about the API endpoints used in the application.

## Example Usage

```bash
# Get Home Content (English)
curl "deramaserver.vercel.app/api/home?lang=en"

# Search for "love"
curl "deramaserver.vercel.app/api/search?q=love&lang=en"

# Get Drama Details (ID: 12345)
curl "deramaserver.vercel.app/api/drama/12345?lang=en"
```

## Endpoints

### 1. Get Languages
Retrieves the list of supported languages.

- **URL:** `/languages`
- **Method:** `GET`
- **Response:** Array of `Language` objects.

```typescript
interface Language {
  code: string;
  name: string;
  path: string;
}
```

### 2. Detect Language
Detects the user's language based on their request (IP/headers).

- **URL:** `/detect-language` (or `/ip`)
- **Method:** `GET`
- **Response:**

```typescript
{
  code: string;
  path: string;
  country: string;
}
```

### 3. Search
Searches for dramas based on a query string. This endpoint performs a global search across all languages by defaulting to English.

- **URL:** `/search`
- **Method:** `GET`
- **Query Parameters:**
  - `q` (string, required): The search query.
  - `lang` (string, optional): Language code (ignored for search, defaults to 'en').
  - `page` (number, optional): Page number (default: 1).
- **Response:**

```typescript
{
  items: ScrapedItem[];
  hasMore: boolean;
  totalPages: number;
}
```

**ScrapedItem Interface:**
```typescript
interface ScrapedItem {
  id: string;
  dramaId?: string;
  title: string;
  link?: string;
  image: string;
  description?: string;
  episode?: string;
  tags?: string[];
}
```

### 4. Browse
Browses dramas by category/type.

- **URL:** `/browse`
- **Method:** `GET`
- **Query Parameters:**
  - `typeId` (string | number, required): The category ID.
  - `page` (number, optional): Page number (default: 1).
  - `lang` (string, optional): Language code (default: 'en'). Note: 'in' is automatically converted to 'id' internally, and 'id' is converted back to 'in' for external requests.
- **Response:**

```typescript
{
  items: ScrapedItem[]; // See ScrapedItem above
  hasNextPage: boolean;
  page: number;
}
```

### 5. Get Home Content
Retrieves content for the home page, including banners and sections.

- **URL:** `/home`
- **Method:** `GET`
- **Query Parameters:**
  - `lang` (string, optional): Language code (default: 'en'). Note: 'in' is automatically converted to 'id' internally, and 'id' is converted back to 'in' for external requests.
- **Response:**

```typescript
{
  banner: ScrapedItem[]; // See ScrapedItem above
  sections: ScrapedSection[];
}
```

**ScrapedSection Interface:**
```typescript
interface ScrapedSection {
  id: string;
  title: string;
  browseTypeId?: number | string; // Pass this ID to /browse endpoint to see more items
  items: ScrapedItem[];
}
```

### 5.1 Get Section Content (See More)
To view all items or paginate through a specific section found on the Home page (e.g., "Trending Now", "Must Watch"), use the **Browse** endpoint.

- **Endpoint:** `/browse`
- **Parameter:** Set `typeId` to the `browseTypeId` value found in the Home section object.
- **Example:**
  If `/home` returns a section with `"browseTypeId": "channel:123"`, call:
  `GET /browse?typeId=channel:123&page=1`

### 6. Get Drama Details
Retrieves detailed information about a specific drama.

- **URL:** `/drama/:id`
- **Method:** `GET`
- **URL Parameters:**
  - `id` (string, required): The unique ID of the drama.
- **Query Parameters:**
  - `lang` (string, optional): Language code (default: 'en'). Note: 'in' is automatically converted to 'id' internally, and 'id' is converted back to 'in' for external requests.
- **Response:** `DramaDetail` object.

**DramaDetail Interface:**
```typescript
interface DramaDetail {
  id: string;
  title: string;
  cover: string;
  introduction: string;
  viewCount: number;
  followCount: number;
  episodeCount: number;
  tags: string[];
  genre?: string;
  cast?: { name: string; avatar?: string; id?: string }[];
  episodes: DramaEpisode[];
  recommendations: ScrapedItem[];
}

interface DramaEpisode {
  id: string;
  index: number;
  cover: string;
  duration: number;
}
```

### 7. Get Watch Episodes
Retrieves episode details for watching a specific drama/book.

- **URL:** `/watch`
- **Method:** `GET`
- **Query Parameters:**
  - `bookId` (string, required): The ID of the book/drama.
  - `lang` (string, optional): Language code (default: 'en'). Note: 'in' is automatically converted to 'id' internally, and 'id' is converted back to 'in' for external requests.
- **Response:** `WatchResponse` object (or array of `WatchEpisode` in some contexts).

**WatchResponse Interface:**
```typescript
interface WatchResponse {
  title: string;
  introduction: string;
  episodes: WatchEpisode[];
}

interface WatchEpisode {
  id: string;
  index: number;
  name: string;
  cover: string;
  duration: number;
  cdnList?: WatchCdn[];
  spriteUrl?: string;
  videoUrl?: string;
  subtitle?: SubtitleSource;
}

interface WatchCdn {
  cdn?: string;
  default?: number;
  videoList?: {
    quality?: number;
    url?: string;
    default?: number;
  }[];
}

type SubtitleSource = string | { url: string }[];
```

### 8. Get Sitemaps
Retrieves a list of available sitemaps.

- **URL:** `/sitemaps`
- **Method:** `GET`
- **Response:**

```typescript
{
  name: string;
  url: string;
  lastModified: string;
  size: string;
}[]
```
