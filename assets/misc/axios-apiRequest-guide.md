# API Request Function Guide

## Table of Contents
- [1. GET Requests](#1-get-requests)
- [2. POST Requests](#2-post-requests)
- [3. PUT Requests](#3-put-requests)
- [4. DELETE Requests](#4-delete-requests)
- [5. PATCH Requests](#5-patch-requests)

This document describes various ways to use the `apiRequest` utility function for different HTTP methods.

## 1. GET Requests

### Simple GET Request
```typescript
import apiRequest from './path/to/apiRequest';

async function fetchData() {
  const response = await apiRequest('GET', '/api/data');
  if (response.success) {
    console.log(response.data);
  } else {
    console.error('Error fetching data:', response.error);
  }
}
```

### GET Request with Query Parameters
```typescript
async function fetchDataWithParams() {
  const params = { search: 'test', limit: 10 };
  const response = await apiRequest<{ items: string[] }>(
    'GET',
    '/api/items',
    null,
    params
  );
  if (response.success) {
    console.log('Fetched items:', response.data?.items);
  } else {
    console.error('Error fetching items:', response.error);
  }
}
```

### GET Request with Server-Side Authentication
```typescript
export async function getServerSideProps(): Promise<{ props: any }> {
  const response = await apiRequest(
    'GET',
    '/api/data',
    null,
    null,
    true
  );
  if (response.success) {
    return { props: { data: response.data } };
  } else {
    return { props: { error: response.error } };
  }
}
```

## 2. POST Requests

### Simple POST Request
```typescript
async function postData() {
  const payload = { name: 'John', age: 30 };
  const response = await apiRequest(
    'POST',
    '/api/submit',
    payload
  );
  if (response.success) {
    console.log('Data submitted successfully');
  } else {
    console.error('Error submitting data:', response.error);
  }
}
```

### POST Request with Form Data
```typescript
async function postFormData(fileInput: HTMLInputElement) {
  const formData = new FormData();
  if (fileInput.files?.[0]) {
    formData.append('file', fileInput.files[0]);
  }
  const response = await apiRequest(
    'POST',
    '/api/upload',
    formData
  );
  if (response.success) {
    console.log('File uploaded successfully');
  } else {
    console.error('Error uploading file:', response.error);
  }
}
```

## 3. PUT Requests

### Simple PUT Request
```typescript
async function updateData() {
  const payload = { id: 1, name: 'Updated Name' };
  const response = await apiRequest(
    'PUT',
    '/api/update',
    payload
  );
  if (response.success) {
    console.log('Data updated successfully');
  } else {
    console.error('Error updating data:', response.error);
  }
}
```

## 4. DELETE Requests

### Simple DELETE Request
```typescript
async function deleteData() {
  const response = await apiRequest(
    'DELETE',
    '/api/delete/1'
  );
  if (response.success) {
    console.log('Data deleted successfully');
  } else {
    console.error('Error deleting data:', response.error);
  }
}
```

## 5. PATCH Requests

### Simple PATCH Request
```typescript
async function patchData() {
  const payload = { name: 'New Name' };
  const response = await apiRequest(
    'PATCH',
    '/api/patch/1',
    payload
  );
  if (response.success) {
    console.log('Data patched successfully');
  } else {
    console.error('Error patching data:', response.error);
  }
}
```
---

## Contribution
Want to suggest axios config options? Open a pull request!

