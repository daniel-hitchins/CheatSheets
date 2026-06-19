---
layout: default
title: Accessibility Developer Cheat Sheet
---

# Accessibility Developer Cheat Sheet
## PlatformUI & Mosaic Design System

> A quick accessibility reference for developers

<details open markdown="1">
<summary><strong>Table of Contents</strong></summary>

- [Quick Start: The Golden Rules](#quick-start-the-golden-rules)
- [Page Structure](#page-structure)
- [Form Controls](#form-controls)
- [Buttons and Links](#buttons-and-links)
- [Images and Icons](#images-and-icons)
- [Navigation and Landmarks](#navigation-and-landmarks)
- [Tables](#tables)
- [Dialogs and Overlays](#dialogs-and-overlays)
- [Loading States](#loading-states)
- [File Uploads](#file-uploads)
- [Dynamic Content](#dynamic-content)
- [Color and Contrast](#color-and-contrast)
- [Keyboard Navigation](#keyboard-navigation)
- [Common Mosaic Components Accessibility Notes](#common-mosaic-components-accessibility-notes)
- [Testing Checklist](#testing-checklist)
- [Quick Reference: ARIA Attributes](#quick-reference-aria-attributes)
- [Common Mistakes to Avoid](#common-mistakes-to-avoid)
- [Getting Help](#getting-help)

</details>

---

## Quick Start: The Golden Rules

1. **Use Mosaic components by default** - They handle accessibility for you
2. **Always provide labels** - Every input needs a visible label
3. **Test with keyboard only** - Can you use your feature without a mouse?
4. **Use semantic HTML** - `<button>` not `<div onclick>`
5. **Add alt text** - Describe images for screen readers

---

## Page Structure

### Page Titles

Every page in PlatformUI needs a descriptive `<title>` element. Screen readers announce page titles when the page loads, and they appear in browser tabs and bookmarks.

```vue
<script setup>
useHead({
  title: 'Edit User - Identity Management'
})
</script>
```

**Format guidelines:**
- `[Page Name] - [Section] - [App Name]`
- For multi-step flows: `Step 2 of 3: Review - Create Application`
- Keep under 60 characters for SEO

**Why:** Helps users understand where they are, especially when switching tabs or using screen readers.

### Heading Hierarchy

Use proper heading levels to create a logical document outline:

```vue
<h1>User Management</h1>  <!-- Page title - only one per page -->

<h2>Active Users</h2>      <!-- Main sections -->
<h3>Admin Users</h3>       <!-- Subsections -->
<h3>Standard Users</h3>

<h2>Pending Invitations</h2>
```

**Rules:**
- One `<h1>` per page (usually the page title)
- Don't skip levels (h1 → h3)
- Nest headings logically

**Why:** Screen reader users navigate by headings to scan page structure.

---

## Form Controls

### Labels Are Required

**❌ Bad:**
```vue
<InputText v-model="email" placeholder="Email" />
```

**✅ Good:**
```vue
<Label for="email">Email</Label>
<InputText id="email" v-model="email" />
```

**Why:** Screen readers need labels to announce what the field is for. Placeholders disappear and aren't labels.

### Help Text and Validation

```vue
<Label for="password">Password</Label>
<InputText id="password" v-model="password" aria-describedby="password-help" />
<HelpText id="password-help">Must be at least 8 characters</HelpText>
```

**Why:** The `aria-describedby` connects the help text to the input for screen readers.

### Password Requirements

For password fields with multiple requirements, link them all:

```vue
<Label for="new-password">New Password</Label>
<Password 
  id="new-password" 
  v-model="password" 
  aria-describedby="password-requirements"
  toggleMask
/>
<HelpText id="password-requirements">
  Must include: 8+ characters, uppercase letter, lowercase letter, and number
</HelpText>
```

**Why:** Screen readers read the requirements when the field receives focus.

### Required Fields

Mark required fields both visually and programmatically:

```vue
<Label for="email">
  Email <abbr title="required" aria-label="required">*</abbr>
</Label>
<InputText 
  id="email" 
  v-model="email" 
  required 
  aria-required="true"
/>
```

**Alternative (better UX):** Add "(required)" in the label text:
```vue
<Label for="email">Email (required)</Label>
<InputText id="email" v-model="email" required aria-required="true" />
```

**Why:** Visual asterisks alone don't communicate to screen readers. The `aria-required` attribute announces the field as required.

### Checkboxes and Radio Buttons

```vue
<!-- Single checkbox -->
<Checkbox id="terms" v-model="accepted" binary />
<Label for="terms">I accept the terms</Label>

<!-- Radio button group -->
<fieldset>
  <legend>Choose a plan</legend>
  <div>
    <RadioButton id="basic" name="plan" value="basic" v-model="selectedPlan" />
    <Label for="basic">Basic</Label>
  </div>
  <div>
    <RadioButton id="pro" name="plan" value="pro" v-model="selectedPlan" />
    <Label for="pro">Pro</Label>
  </div>
</fieldset>
```

**Why:** 
- Labels must come AFTER checkboxes/radios in the DOM
- Group related radio buttons in a `<fieldset>` with a `<legend>`
- Use the same `name` for radio buttons in a group

---

## Buttons and Links

### Use the Right Element

**❌ Bad:**
```vue
<div @click="save">Save</div>
<a @click="openDialog">Open</a>
```

**✅ Good:**
```vue
<Button @click="save">Save</Button>
<Button @click="openDialog" text>Open</Button>
```

**Why:** 
- `<button>` is keyboard accessible by default
- Screen readers announce it as a button
- Links (`<a>`) are for navigation to URLs only

### Button Labels

```vue
<!-- Icon buttons need labels -->
<Button icon="pi pi-times" aria-label="Close" text />

<!-- Or use a Tooltip for visual users -->
<Button icon="pi pi-times" aria-label="Delete item" text v-tooltip.top="'Delete'" />
```

**Why:** Icon-only buttons need text labels for screen readers. The `aria-label` provides that.

### Touch Target Sizes

All interactive elements (buttons, links, form controls) must be large enough to tap easily.

**Minimum sizes:**
- **24×24 pixels** (WCAG 2.2 minimum)
- **44×44 pixels** (better for primary actions)

**✅ Mosaic components already meet this requirement.**

**Watch out for:**
- Custom icon-only buttons
- Close buttons in dialog corners
- Inline links with single characters or short words
- Custom checkboxes/radio buttons

**Fix small targets:**
```vue
<!-- ❌ Too small -->
<button class="icon-btn">×</button>

<!-- ✅ Mosaic Button (proper size) -->
<Button icon="pi pi-times" aria-label="Close" text />

<!-- ✅ Add padding to custom elements -->
<a href="#" style="padding: 12px;">Link</a>
```

**Why:** Mobile and touch users need bigger tap areas. Low dexterity users benefit too.

---

## Images and Icons

### Decorative vs Meaningful

**Decorative (no screen reader announcement needed):**
```vue
<Icon name="pi pi-star" aria-hidden="true" />
<span>Featured</span>
```

**Meaningful (describes content):**
```vue
<Avatar :image="user.photo" :label="user.name" alt="Profile picture of Jane Smith" />
```

**Why:** 
- `aria-hidden="true"` tells screen readers to skip decorative icons
- Images that convey information need descriptive `alt` text

---

## Navigation and Landmarks

### Use Semantic HTML Regions

```vue
<header>
  <NavigationHeader><!-- App header --></NavigationHeader>
</header>

<nav aria-label="Main navigation">
  <NavigationRail><!-- Side nav --></NavigationRail>
</nav>

<main>
  <ContentWrapper>
    <!-- Your main content here -->
  </ContentWrapper>
</main>
```

**Why:** Screen readers let users jump to landmarks like `<main>`, `<nav>`, `<header>`. Don't use `<div>` for these.

### Skip Links

```vue
<SkipToRegion targetId="main-content" />

<!-- Later in your template -->
<main id="main-content">
  <!-- Content -->
</main>
```

**Why:** Keyboard users can skip repetitive navigation and jump straight to content.

---

## Tables

### Always Use Headers

```vue
<DataTable :value="users">
  <Column field="name" header="Name"></Column>
  <Column field="email" header="Email"></Column>
  <Column field="role" header="Role"></Column>
</DataTable>
```

**Why:** The `header` prop creates proper `<th>` elements that screen readers announce with each cell.

### Complex Tables

For multi-row headers or complex layouts, ensure proper `scope` attributes:
```vue
<Column field="total" header="Total">
  <template #body="{ data }">
    <span role="cell" aria-label="Total: {{data.total}} dollars">
      ${{data.total}}
    </span>
  </template>
</Column>
```

---

## Dialogs and Overlays

### Focus Management

```vue
<Dialog v-model:visible="showDialog" header="Confirm Delete" modal>
  <p>Are you sure you want to delete this item?</p>
  <template #footer>
    <Button label="Cancel" @click="showDialog = false" text />
    <Button label="Delete" @click="confirmDelete" severity="danger" autofocus />
  </template>
</Dialog>
```

**Why:** 
- Mosaic Dialog automatically traps focus inside the modal
- `autofocus` tells the dialog which button to focus first
- Focus returns to the trigger button when closed

### Announcements

```vue
<Toast />

<!-- In your code -->
toast.add({
  severity: 'success',
  summary: 'Success',
  detail: 'Item deleted successfully',
  life: 3000
});
```

**Why:** Toasts are announced to screen readers via `role="alert"`.

---

## Loading States

### Progress Indicators

```vue
<!-- For known progress -->
<ProgressBar :value="uploadProgress" aria-label="Upload progress" />

<!-- For unknown duration -->
<ProgressSpinner aria-label="Loading data" />
```

**Why:** The `aria-label` tells screen readers what's loading.

### Skeleton Screens

```vue
<Skeleton v-if="loading" width="100%" height="2rem" />
<div v-else>{{ data.title }}</div>
```

**Why:** Better than spinners for content that's about to appear in place.

---

## File Uploads

### Provide Clear Instructions

File upload fields need instructions about accepted formats, size limits, and dimensions:

```vue
<Label for="company-logo">Company Logo</Label>
<FileUpload 
  id="company-logo"
  accept="image/png,image/jpeg"
  :maxFileSize="2000000"
  aria-describedby="logo-requirements"
/>
<HelpText id="logo-requirements">
  PNG or JPG format, maximum 2MB, recommended size 400×400 pixels
</HelpText>
```

**For drag-and-drop uploads:**
```vue
<FileUpload 
  mode="basic"
  accept=".pdf,.doc,.docx"
  :maxFileSize="5000000"
  chooseLabel="Select files or drag here"
  aria-describedby="doc-help"
/>
<HelpText id="doc-help">
  PDF or Word documents, up to 5MB each
</HelpText>
```

**Why:** 
- Users need to know what files are acceptable before selecting
- Prevents upload errors and frustration
- Screen readers announce the requirements when the field receives focus

---

## Dynamic Content

### Live Regions

```vue
<div role="status" aria-live="polite">
  {{ statusMessage }}
</div>
```

**Use cases:**
- `aria-live="polite"` - Announces when screen reader is idle (search results, form validation)
- `aria-live="assertive"` - Announces immediately (errors, urgent alerts)

**Why:** Screen readers don't automatically announce content changes. Live regions tell them to announce updates.

---

## Color and Contrast

### Don't Rely on Color Alone

**❌ Bad:**
```vue
<span style="color: red">Error</span>
```

**✅ Good:**
```vue
<Message severity="error" icon="pi pi-exclamation-triangle">
  Error: Invalid email format
</Message>
```

**Why:** Colorblind users and screen reader users can't perceive color. Use icons and text labels too.

### Contrast Ratios

- **Normal text:** 4.5:1 minimum
- **Large text (18pt+):** 3:1 minimum
- **UI components:** 3:1 minimum

**Tool:** Use browser DevTools contrast checker or [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)

---

## Keyboard Navigation

### Tab Order

- Use native HTML elements (they're already keyboard accessible)
- Don't use positive `tabindex` values (e.g., `tabindex="1"`)
- Use `tabindex="-1"` only when you need to focus something programmatically
- Use `tabindex="0"` to add non-interactive elements to tab order (rare)

### Focus Indicators

**Never do this:**
```css
*:focus {
  outline: none; /* ❌ Bad! */
}
```

**Why:** Focus indicators show keyboard users where they are. Removing them breaks keyboard navigation.

---

## Common Mosaic Components Accessibility Notes

### Accordion
```vue
<Accordion>
  <AccordionPanel header="Section 1">
    Content here
  </AccordionPanel>
</Accordion>
```
✅ Already accessible - uses proper ARIA attributes and keyboard support

### Tabs
```vue
<Tabs>
  <TabList>
    <Tab>Overview</Tab>
    <Tab>Details</Tab>
  </TabList>
  <TabPanels>
    <TabPanel>Overview content</TabPanel>
    <TabPanel>Details content</TabPanel>
  </TabPanels>
</Tabs>
```
✅ Already accessible - arrow keys navigate tabs, proper ARIA roles

### Select/Dropdown
```vue
<Label for="country">Country</Label>
<Select id="country" v-model="selected" :options="countries" optionLabel="name" />
```
✅ Already accessible - keyboard searchable, screen reader support

---

## Testing Checklist

### Manual Testing

1. **Keyboard only:**
   - Can you Tab through all interactive elements?
   - Can you activate buttons with Enter/Space?
   - Can you close dialogs with Escape?
   - Are dropdowns navigable with arrow keys?

2. **Screen reader (NVDA/JAWS/VoiceOver):**
   - Are form labels announced?
   - Are error messages read aloud?
   - Can you navigate by headings?
   - Are images described?

3. **Zoom to 200%:**
   - Does the layout still work?
   - Is all text readable?
   - Do buttons remain clickable?

4. **Zoom to 400% (reflow test):**
   - Press Ctrl/Cmd + (zoom to 400% in browser)
   - No horizontal scrolling should appear
   - Content should reflow to fit the viewport
   - All functionality still works
   - **Note:** Mosaic components handle this automatically, but check custom layouts

### Browser DevTools

- **Chrome:** Lighthouse accessibility audit
- **Firefox:** Accessibility inspector
- **Edge:** Similar to Chrome

---

## Quick Reference: ARIA Attributes

<table>
  <thead>
    <tr>
      <th scope="col">Attribute</th>
      <th scope="col">Purpose</th>
      <th scope="col">Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>aria-label</code></td>
      <td>Provides text label</td>
      <td><code>&lt;button icon="pi pi-times" aria-label="Close" /&gt;</code></td>
    </tr>
    <tr>
      <td><code>aria-labelledby</code></td>
      <td>Points to label element</td>
      <td><code>&lt;div role="dialog" aria-labelledby="title"&gt;</code></td>
    </tr>
    <tr>
      <td><code>aria-describedby</code></td>
      <td>Points to description</td>
      <td><code>&lt;input aria-describedby="help-text"&gt;</code></td>
    </tr>
    <tr>
      <td><code>aria-hidden</code></td>
      <td>Hides from screen readers</td>
      <td><code>&lt;Icon aria-hidden="true" /&gt;</code></td>
    </tr>
    <tr>
      <td><code>aria-live</code></td>
      <td>Announces dynamic changes</td>
      <td><code>&lt;div aria-live="polite"&gt;Status&lt;/div&gt;</code></td>
    </tr>
    <tr>
      <td><code>aria-expanded</code></td>
      <td>Shows expand/collapse state</td>
      <td>Usually handled by Mosaic components</td>
    </tr>
    <tr>
      <td><code>aria-current</code></td>
      <td>Shows current item in nav</td>
      <td><code>&lt;a aria-current="page"&gt;Home&lt;/a&gt;</code></td>
    </tr>
  </tbody>
</table>

---

## Common Mistakes to Avoid

1. ❌ Using `<div>` or `<span>` as buttons
2. ❌ Missing form labels
3. ❌ Placeholder text instead of labels
4. ❌ Removing focus indicators with CSS
5. ❌ Color as the only indicator of state
6. ❌ Empty links or buttons
7. ❌ Images without alt text
8. ❌ Poor heading hierarchy (h1 → h4, skipping h2/h3)
9. ❌ Auto-playing media without controls
10. ❌ Opening new windows without warning
11. ❌ Missing or generic page titles
12. ❌ Touch targets smaller than 24×24px
13. ❌ Not marking required fields programmatically
14. ❌ File uploads without format/size instructions

---

## Getting Help

- **Mosaic Component Docs:** Check component-specific accessibility features
- **WCAG Quick Reference:** https://www.w3.org/WAI/WCAG21/quickref/
- **WebAIM:** https://webaim.org/ (great articles and tools)
- **a11y Project:** https://www.a11yproject.com/ (practical checklist)

---

## Remember

> "Accessibility is not a feature. It's a requirement."

Most accessibility issues are fixed by:
1. Using semantic HTML
2. Using Mosaic components correctly
3. Always adding labels
4. Testing with a keyboard

Start with these fundamentals, and you'll handle 80% of accessibility concerns.
