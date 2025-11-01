<!--
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2025 The Linux Foundation
-->

# 🌍 Check URL for Validity with WGET

Checks if a remote web server returns a valid page for a given URL.

## url-validity-action

## Usage Examples

### Basic Usage

<!-- markdownlint-disable MD013 -->

```yaml
steps:
  - name: 'Check remote web server for URL'
    uses: lfreleng-actions/url-validity-action@main
    with:
      url: "https://linuxfoundation.org"
```

### Advanced Usage

```yaml
steps:
  - name: 'Check remote web server for URL'
    id: url-check
    uses: lfreleng-actions/url-validity-action@main
    with:
      url: "https://linuxfoundation.org"

  - name: 'Process results'
    run: |
      echo "URL is valid: ${{ steps.url-check.outputs.valid }}"
      echo "HTTP Status Code: ${{ steps.url-check.outputs.http_code }}"
      if [ -n "${{ steps.url-check.outputs.redirect_url }}" ]; then
        echo "Redirected to: ${{ steps.url-check.outputs.redirect_url }}"
        echo "Redirect is valid: ${{ steps.url-check.outputs.redirect_valid }}"
        echo "Redirect loop detected: ${{ steps.url-check.outputs.redirect_loop }}"
      fi

  - name: 'Handle different scenarios'
    run: |
      # Check for redirect loops first
      if [ "${{ steps.url-check.outputs.redirect_loop }}" = "true" ]; then
        echo "🔄 Redirect loop detected - URLs are redirecting in a cycle"
        exit 1
      fi

      # Handle different HTTP status codes
      case "${{ steps.url-check.outputs.http_code }}" in
        200)
          echo "✅ Success - URL is accessible"
          ;;
        301|302|303|307|308)
          echo "🔄 Redirect - Final URL: ${{ steps.url-check.outputs.redirect_url }}"
          if [ "${{ steps.url-check.outputs.redirect_valid }}" = "true" ]; then
            echo "✅ Redirected URL is valid"
          else
            echo "❌ Redirected URL is invalid"
          fi
          ;;
        404)
          echo "❌ Not Found - URL does not exist"
          ;;
        500|502|503|504)
          echo "🚫 Server Error - Remote server is experiencing issues"
          ;;
        *)
          echo "⚠️ Unexpected HTTP code: ${{ steps.url-check.outputs.http_code }}"
          ;;
      esac
```

<!-- markdownlint-enable MD013 -->

## Implementation

Uses the shell command:

`wget [URL] --spider --server-response --max-redirect=10`

This enhanced version captures HTTP status codes and follows up to 10 redirects
to determine the final destination URL.

## Inputs

<!-- markdownlint-disable MD013 -->

| Input | Required | Description  | Example                       |
| ----- | -------- | ------------ | ----------------------------- |
| url   | true     | URL to check | <https://linuxfoundation.org> |

<!-- markdownlint-enable MD013 -->

## Outputs

<!-- markdownlint-disable MD013 -->

| Variable Name  | Mandatory | Description                                                             |
| -------------- | --------- | ----------------------------------------------------------------------- |
| valid          | True      | Set true or false based on WGET exit status                             |
| http_code      | False     | HTTP status code returned by the server (e.g., 200, 404, 500)           |
| redirect_url   | False     | Final URL after following redirects (empty if no redirects)             |
| redirect_valid | False     | True if redirected URL is valid, false if invalid, empty if no redirect |
| redirect_loop  | False     | True if redirect loop detected, false otherwise                         |

<!-- markdownlint-enable MD013 -->
