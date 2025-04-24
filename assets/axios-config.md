# Axios Setup for API Requests

This document outlines the Axios setup for handling API requests, including a global instance, error handling, and authentication.

## Installation

Ensure you have Axios installed in your project:

```bash
pnpm install axios
```

## Axios Instance

The `axiosInstance` is configured with a base URL and a timeout limit of 30 seconds.

```typescript
import axios, { Method } from 'axios';

const axiosInstance = axios.create({
  timeout: 30000,
  baseURL: process.env.BASE_URL,
});
```

### Automatic Retry Mechanism

If a request fails due to server errors (e.g., `500`, `503`), the request is retried once after a 1-second delay.

```typescript
axiosInstance.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (
      !error.config._retry &&
      [408, 429, 500, 502, 503, 504].includes(error.response?.status || 0)
    ) {
      error.config._retry = true;
      await new Promise((res) => setTimeout(res, 1000));
      return axiosInstance(error.config);
    }
    return Promise.reject(error);
  },
);
```

## Authentication Handling

For server-side requests, Basic Authentication is supported using environment variables.

```typescript
const getAuth = (): string | null => {
  if (typeof window !== 'undefined') return null;
  const { AUTH_USERNAME, AUTH_PASSWORD } = process.env;
  return AUTH_USERNAME && AUTH_PASSWORD
    ? `Basic ${Buffer.from(`${AUTH_USERNAME}:${AUTH_PASSWORD}`).toString('base64')}`
    : null;
};
```

## API Request Function

The `apiRequest` function allows sending requests with dynamic methods, headers, and parameters.

```typescript
type ApiResponse<K> = {
  success: boolean;
  status?: number;
  error?: string;
  data: K | null;
};

const apiRequest = async <K>(
  method: Method,
  url: string,
  data: unknown = null,
  params: Record<string, unknown> | null = null,
  isServerSide: boolean = false,
): Promise<ApiResponse<K>> => {
  try {
    const headers: Record<string, string> = {};
    if (isServerSide && typeof window === 'undefined') {
      const auth = getAuth();
      if (auth) headers.Authorization = auth;
    }
    if (!(data instanceof FormData))
      headers['Content-Type'] = 'application/json';

    const response = await axiosInstance({
      method,
      url,
      data,
      params,
      headers,
    });
    return { success: true, data: response.data };
  } catch (error) {
    return {
      success: false,
      status:
        (error as { response: { status: number } }).response?.status || 500,
      error: (error as { message: string }).message,
      data: (error as { response: { data: K } }).response?.data || null,
    };
  }
};

export default apiRequest;
```

## Full Code

```typescript
import axios, { Method } from "axios";

const API_BASE_URL = process.env.NEXT_PUBLIC_API_BASE_URL;
const API_USERNAME = process.env.API_USERNAME;
const API_PASSWORD = process.env.API_PASSWORD;


// Only server-side: Create Basic Auth Header
const getAuthHeader = (): string | undefined => {
  if (typeof window !== "undefined") return undefined;
  if (!API_USERNAME || !API_PASSWORD) return undefined;

  return `Basic ${Buffer.from(`${API_USERNAME}:${API_PASSWORD}`).toString("base64")}`;
};

const axiosInstance = axios.create({
  baseURL: API_BASE_URL,
  timeout: 30000,
});

axiosInstance.interceptors.response.use(
  (response) => response,
  async (error) => {
    const retryStatusCodes = [408, 429, 500, 502, 503, 504];
    const config = error.config;

    if (!config._retry && retryStatusCodes.includes(error?.response?.status)) {
      config._retry = true;
      await new Promise((res) => setTimeout(res, 1000));
      return axiosInstance(config);
    }

    return Promise.reject(error);
  },
);

export type ApiResponseType<K> = {
  success: boolean;
  status?: number;
  error?: string;
  data: K | null;
};

// Main API requester
export const apiRequest = async <K>(
  method: Method,
  url: string,
  data: unknown = null,
  params: Record<string, unknown> | null = null,
): Promise<ApiResponseType<K>> => {
  try {
    const headers: Record<string, string> = {};

    const auth = getAuthHeader();
    if (auth) headers["Authorization"] = auth;

    if (!(data instanceof FormData)) {
      headers["Content-Type"] = "application/json";
    }

    const response = await axiosInstance({
      method,
      url,
      data,
      params,
      headers,
    });

    return { success: true, data: response.data };
  } catch (err: any) {
    return {
      success: false,
      status: err.response?.status ?? 500,
      error: err.response?.data?.detail || err.message || "API request failed",
      data: err.response?.data ?? null,
    };
  }
};

export default apiRequest;
```

For more details, check out the [API Request Function Guide](./misc/axios-apiRequest-guide.md)

---

## Contribution
Want to suggest axios config options? Open a pull request!
