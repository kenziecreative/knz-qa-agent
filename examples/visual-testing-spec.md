# E-Commerce Homepage

## Overview

The homepage is the primary landing page for the e-commerce store. It features a hero banner, product grid, navigation, and footer. This spec demonstrates the v2 visual testing features: Visual Focus, Design Reference, and Browsers.

## Base URL

http://localhost:3000

## Personas

- unauthenticated user (browsing without login)
- logged-in user (use test@example.com / password123)

## Viewports

- desktop (1280x720)
- mobile (375x667)

## Visual Focus

- design-verification
- layout-integrity

## Design Reference

### desktop
- home-logged-out: .qa/designs/desktop-home-logged-out.png
- home-logged-in: .qa/designs/desktop-home-logged-in.png

### mobile
- home-logged-out: .qa/designs/mobile-home-logged-out.png

## Browsers

- chromium
- webkit

## Test Scenarios

### 1. Homepage Loads Correctly

tags: [smoke, critical]

**Steps:**

1. Navigate to /
2. Verify the page loads without console errors
3. Check that the hero banner is visible
4. Check that the product grid displays at least 4 products

**Expected:**

- Hero banner image loads and is fully visible
- Product grid shows product cards with images, titles, and prices
- Navigation bar is present with all expected links
- Footer is visible at the bottom
- No console errors or warnings
- Page loads within 2 seconds

### 2. Responsive Layout

tags: [regression]
depends_on: 1. Homepage Loads Correctly

**Steps:**

1. At desktop viewport, verify the product grid shows 4 columns
2. At mobile viewport, verify the product grid shows 1-2 columns
3. At mobile viewport, verify the navigation collapses to a hamburger menu

**Expected:**

- Desktop: 4-column product grid, full navigation bar
- Mobile: 1-2 column product grid, hamburger menu
- No horizontal scrolling at any viewport
- All images scale appropriately without distortion

## Edge Cases to Verify

- Very long product names (50+ characters) should truncate or wrap without breaking layout
- Missing product images should show a placeholder, not a broken image icon
- Empty product grid (no products) should show a meaningful empty state

## Things to Watch For

- **Performance**: Largest Contentful Paint should be under 2.5s
- **Visual**: Product card spacing should be consistent across the grid
- **Accessibility**: All images should have alt text, navigation should be keyboard-accessible
- **Cross-browser**: Hero banner rendering may differ between Chromium and WebKit — note any differences

## Known Issues

None currently tracked.
