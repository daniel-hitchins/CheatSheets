---
layout: default
title: Risks Accessibility Developer Cheat Sheet
---

# Risks Accessibility Developer Cheat Sheet
## PlatformUI & Mosaic Design System

> A quick accessibility reference for developers

## Quick Start: The Golden Rules

1. **Use Mosaic components by default** - They handle accessibility for you
2. **Charts need text alternatives** - Describe the data, not just "chart"
3. **Don't rely on color alone** - Use patterns, labels, and icons too
4. **Make everything keyboard accessible** - Charts, filters, tables, everything
5. **Test with screen readers** - NVDA/JAWS/VoiceOver should make sense

---

## Page Structure

### Page Titles

Every page in the Risks product needs a descriptive title that identifies the current view or step.

```vue
<script setup>
// ✅ Specific to current page
useHead({
  title: 'Risk Assessment - Client Name - Risks'
});

// ✅ For multi-step flows
useHead({
  title: 'Step 2 of 3: Review Controls - Risk Assessment'
});

// ❌ Too generic
useHead({
  title: 'Risks'
});
</script>
```

**Format guidelines:**
- `[View Name] - [Context] - Risks`
- Include client/assessment name when relevant
- Update dynamically as user navigates

**Why:** Users with multiple tabs open need to know which risk view they're looking at.

### Headings and Labels

Use descriptive, specific headings - not generic ones.

**❌ Too generic:**
```vue
<h2>Details</h2>
<h2>Information</h2>
<h2>Data</h2>
```

**✅ Specific and clear:**
```vue
<h2>Risk Controls Breakdown</h2>
<h2>Assessment Timeline</h2>
<h2>High Priority Risks</h2>
```

### Remove Empty Headings

**❌ Bad:**
```vue
<h2></h2>
<h3>{{ }}</h3>
<h2 v-if="false">Title</h2>
```

**✅ Good:**
```vue
<!-- Only render heading if there's content -->
<h2 v-if="section.title">{{ section.title }}</h2>

<!-- Or provide default content -->
<h2>{{ section.title || 'Untitled Section' }}</h2>
```

**Why:** Empty headings confuse screen reader users who navigate by heading structure.

---

## Buttons and Interactive Elements

### Accessible Names

Every button and link needs a clear name that describes its action.

**❌ Missing or unclear names:**
```vue
<Button icon="pi pi-times" />
<Button icon="pi pi-pencil" />
<a href="/details"><Icon name="pi pi-info-circle" /></a>
```

**✅ Clear accessible names:**
```vue
<Button icon="pi pi-times" aria-label="Remove filter" />
<Button icon="pi pi-pencil" aria-label="Edit risk assessment" />
<a href="/details" aria-label="View risk details">
  <Icon name="pi pi-info-circle" aria-hidden="true" />
</a>
```

### Icon Indicators

Movement icons and status indicators need text alternatives.

**❌ Icon only:**
```vue
<span class="risk-trend">
  <Icon name="pi pi-arrow-up" :style="{ color: 'red' }" />
</span>
```

**✅ Icon with text:**
```vue
<span class="risk-trend">
  <Icon name="pi pi-arrow-up" aria-hidden="true" />
  <span class="sr-only">Risk increased</span>
</span>

<!-- Or use aria-label on container -->
<span class="risk-trend" aria-label="Risk increased from medium to high">
  <Icon name="pi pi-arrow-up" aria-hidden="true" />
</span>
```

**Icon font indicators:**
```vue
<!-- ❌ Icon font without label -->
<i class="ri-alert-fill"></i>

<!-- ✅ With accessible label -->
<i class="ri-alert-fill" role="img" aria-label="High priority"></i>

<!-- ✅ Or use Mosaic Badge -->
<Badge severity="danger" value="High Priority" />
```

---

## Form Validation

### Clear Error Messages

Be specific about what's wrong and how to fix it.

**❌ Generic error:**
```vue
<ValidationText>Invalid input</ValidationText>
<ValidationText>Error</ValidationText>
<ValidationText>Please check your entry</ValidationText>
```

**✅ Specific error:**
```vue
<ValidationText>Risk score must be between 1 and 100</ValidationText>
<ValidationText>Assessment date cannot be in the future</ValidationText>
<ValidationText>At least one control must be selected</ValidationText>
```

### Error Identification

Link errors to their fields and announce them to screen readers.

```vue
<Label for="risk-score">Risk Score (required)</Label>
<InputNumber 
  id="risk-score" 
  v-model="score"
  :class="{ 'p-invalid': errors.score }"
  aria-describedby="risk-score-error"
  aria-invalid="true"
/>
<ValidationText id="risk-score-error" v-if="errors.score">
  {{ errors.score }}
</ValidationText>
```

**For multiple errors on a page:**
```vue
<Message v-if="hasErrors" severity="error" role="alert">
  <strong>Please fix the following errors:</strong>
  <ul>
    <li><a href="#risk-score">Risk score is required</a></li>
    <li><a href="#control-type">Control type must be selected</a></li>
  </ul>
</Message>
```

**Why:** Screen readers announce errors via `role="alert"` and users can navigate directly to problem fields.

---

## Data Visualization Accessibility

### Charts - Text Alternatives

Every chart needs a text description that conveys the same information as the visual.

**❌ Generic alt text:**
```vue
<div role="img" aria-label="Chart">
  <PieChart :data="riskData" />
</div>
```

**✅ Descriptive alternative:**
```vue
<figure role="figure" aria-labelledby="risk-chart-title">
  <figcaption id="risk-chart-title" class="sr-only">
    Risk Distribution by Severity: 
    Critical: 12 (15%), 
    High: 28 (35%), 
    Medium: 32 (40%), 
    Low: 8 (10%)
  </figcaption>
  <PieChart :data="riskData" aria-hidden="true" />
</figure>

<!-- Provide data table as alternative -->
<details>
  <summary>View data table</summary>
  <DataTable :value="riskData">
    <Column field="severity" header="Severity"></Column>
    <Column field="count" header="Count"></Column>
    <Column field="percentage" header="Percentage"></Column>
  </DataTable>
</details>
```

### Pie Charts and Donut Charts

**Color alone is not enough:**

**❌ Color-only differentiation:**
```vue
<PieChart 
  :data="risksBySeverity"
  :colors="['#red', '#orange', '#yellow', '#green']"
/>
```

**✅ Color + Patterns + Labels:**
```vue
<PieChart 
  :data="risksBySeverity"
  :colors="['#dc3545', '#fd7e14', '#ffc107', '#28a745']"
  showLabels
  showValues
/>

<!-- Add pattern fills for color-blind users -->
<PieChart 
  :data="risksBySeverity"
  :patterns="['diagonal', 'dots', 'horizontal', 'solid']"
  showLegend
/>
```

**Always provide:**
1. **Legend** with text labels
2. **Data labels** on segments
3. **Percentage values** 
4. **Data table** alternative

### Heatmap Grids

Heatmaps are complex and need special treatment.

**Keyboard navigation:**
```vue
<div 
  class="heatmap-grid"
  role="grid"
  aria-label="Risk heatmap: Likelihood vs Impact"
  aria-describedby="heatmap-description"
>
  <div id="heatmap-description" class="sr-only">
    Navigate with arrow keys. Each cell shows risk count for likelihood and impact combination.
  </div>
  
  <div role="row" v-for="likelihood in likelihoods" :key="likelihood">
    <div role="rowheader">{{ likelihood }}</div>
    <div 
      v-for="impact in impacts" 
      :key="impact"
      role="gridcell"
      :aria-label="`${likelihood} likelihood, ${impact} impact: ${getCount(likelihood, impact)} risks`"
      :tabindex="0"
      @click="selectCell(likelihood, impact)"
      @keydown.enter="selectCell(likelihood, impact)"
      @keydown.space.prevent="selectCell(likelihood, impact)"
      :style="{ backgroundColor: getCellColor(likelihood, impact) }"
    >
      <span aria-hidden="true">{{ getCount(likelihood, impact) }}</span>
    </div>
  </div>
</div>
```

**Color + Pattern:**
```vue
<div 
  :class="[
    'heatmap-cell',
    `risk-${riskLevel}`, // CSS class for background color
    `pattern-${riskLevel}` // CSS class for pattern overlay
  ]"
  :aria-label="cellDescription"
>
  {{ riskCount }}
</div>

<style>
/* Use both color and pattern */
.risk-critical {
  background-color: #dc3545;
  background-image: repeating-linear-gradient(45deg, transparent, transparent 10px, rgba(255,255,255,.3) 10px, rgba(255,255,255,.3) 20px);
}

.risk-high {
  background-color: #fd7e14;
  background-image: radial-gradient(circle, rgba(255,255,255,.3) 25%, transparent 25%);
}
</style>
```

**Why:** Color-blind users need patterns. Keyboard users need proper grid roles and navigation.

### Donut Chart Segments

Make individual segments keyboard accessible:

```vue
<svg role="img" aria-labelledby="donut-title">
  <title id="donut-title">Risk Controls Breakdown</title>
  
  <g 
    v-for="(segment, index) in segments" 
    :key="index"
    role="button"
    :tabindex="0"
    :aria-label="`${segment.label}: ${segment.value} controls, ${segment.percentage}%`"
    @click="selectSegment(segment)"
    @keydown.enter="selectSegment(segment)"
    @keydown.space.prevent="selectSegment(segment)"
  >
    <path :d="segment.path" :fill="segment.color" />
  </g>
</svg>
```

---

## Interactive Tables

### Keyboard Navigation

Table rows need to be keyboard accessible:

**❌ Click-only rows:**
```vue
<DataTable :value="risks">
  <Column field="id" header="ID"></Column>
  <Column>
    <template #body="{ data }">
      <div @click="viewRisk(data.id)">View</div>
    </template>
  </Column>
</DataTable>
```

**✅ Keyboard accessible rows:**
```vue
<DataTable 
  :value="risks"
  selectionMode="single"
  @row-select="onRowSelect"
  :rowClass="rowClass"
>
  <Column field="id" header="Risk ID"></Column>
  <Column field="title" header="Title"></Column>
  <Column>
    <template #body="{ data }">
      <Button 
        label="View Details" 
        @click="viewRisk(data.id)"
        @keydown.enter="viewRisk(data.id)"
        text
      />
    </template>
  </Column>
</DataTable>

<script setup>
const rowClass = (data) => {
  return { 'cursor-pointer': true };
};
</script>
```

**Why:** Mosaic DataTable handles most keyboard navigation, but custom actions need explicit keyboard support.

---

## Filter Chips

### Remove Buttons

Filter chips need keyboard-accessible remove buttons:

**❌ Mouse-only remove:**
```vue
<Chip 
  :label="filter.label"
  @remove="removeFilter(filter)"
/>
```

**✅ Keyboard accessible:**
```vue
<Chip 
  :label="filter.label"
  :removable="true"
  @remove="removeFilter(filter)"
  :removeButtonProps="{
    'aria-label': `Remove ${filter.label} filter`
  }"
/>

<!-- Or custom implementation -->
<div class="filter-chip">
  <span>{{ filter.label }}</span>
  <Button
    icon="pi pi-times"
    :aria-label="`Remove ${filter.label} filter`"
    @click="removeFilter(filter)"
    @keydown.enter="removeFilter(filter)"
    text
    rounded
    size="small"
  />
</div>
```

**Announce changes:**
```vue
<script setup>
const removeFilter = (filter) => {
  filters.value = filters.value.filter(f => f.id !== filter.id);
  
  // Announce to screen readers
  announceToScreenReader(`${filter.label} filter removed`);
};

const announceToScreenReader = (message) => {
  const announcement = document.createElement('div');
  announcement.setAttribute('role', 'status');
  announcement.setAttribute('aria-live', 'polite');
  announcement.textContent = message;
  announcement.classList.add('sr-only');
  document.body.appendChild(announcement);
  
  setTimeout(() => {
    document.body.removeChild(announcement);
  }, 1000);
};
</script>
```

---

## Focus Management

### Tab Components

Ensure proper focus order in tab navigation:

**❌ Wrong focus order:**
```vue
<!-- Focus jumps around confusingly -->
<Tabs>
  <TabList>
    <Tab>Overview</Tab>
    <Tab>Controls</Tab>
  </TabList>
  <Button>Action</Button> <!-- ❌ Button in between tabs and panels -->
  <TabPanels>
    <TabPanel>Overview content</TabPanel>
    <TabPanel>Controls content</TabPanel>
  </TabPanels>
</Tabs>
```

**✅ Correct focus order:**
```vue
<Tabs>
  <TabList>
    <Tab>Overview</Tab>
    <Tab>Controls</Tab>
  </TabList>
  <TabPanels>
    <TabPanel>
      Overview content
      <Button>Action in Overview</Button>
    </TabPanel>
    <TabPanel>
      Controls content
      <Button>Action in Controls</Button>
    </TabPanel>
  </TabPanels>
</Tabs>
```

**Why:** Focus should flow: tabs → active panel content → next tab/panel.

### AI Suggestions Modal

Focus should move into the modal and return properly:

```vue
<Dialog 
  v-model:visible="showAISuggestions"
  header="AI Risk Suggestions"
  modal
  :closable="true"
  @show="onModalOpen"
  @hide="onModalClose"
>
  <div ref="modalContent">
    <p>AI has identified the following risks:</p>
    <ul>
      <li v-for="suggestion in suggestions" :key="suggestion.id">
        {{ suggestion.text }}
      </li>
    </ul>
  </div>
  
  <template #footer>
    <Button 
      label="Dismiss" 
      @click="showAISuggestions = false"
      text
    />
    <Button 
      label="Apply Suggestions" 
      @click="applySuggestions"
      autofocus
    />
  </template>
</Dialog>

<script setup>
const triggerButton = ref(null);

const onModalOpen = () => {
  // Store reference to button that opened modal
  triggerButton.value = document.activeElement;
};

const onModalClose = () => {
  // Return focus to trigger button
  nextTick(() => {
    triggerButton.value?.focus();
  });
};
</script>
```

### Drawers

Same principle for side drawers:

```vue
<Drawer 
  v-model:visible="showFilters"
  header="Filter Risks"
  position="right"
  @show="onDrawerOpen"
  @hide="onDrawerClose"
>
  <!-- Filter content -->
  <template #footer>
    <Button label="Clear" @click="clearFilters" text />
    <Button label="Apply" @click="applyFilters" />
  </template>
</Drawer>
```

**Why:** Mosaic Dialog and Drawer handle most of this, but you need to manage focus return for custom triggers.

---

## Markup Validity

### Button Structure

Buttons can't contain interactive elements:

**❌ Invalid markup:**
```vue
<Button>
  <a href="/details">View Details</a> <!-- ❌ Link inside button -->
</Button>

<Button>
  <Button icon="pi pi-times" /> <!-- ❌ Button inside button -->
</Button>
```

**✅ Valid markup:**
```vue
<!-- Use button for actions -->
<Button @click="navigateTo('/details')">View Details</Button>

<!-- Or link styled as button -->
<a href="/details" class="p-button">View Details</a>

<!-- Button group for multiple actions -->
<ButtonGroup>
  <Button label="View" @click="view" />
  <Button icon="pi pi-times" @click="close" />
</ButtonGroup>
```

### Grid Structure (Heatmap)

Grids need proper role structure:

**❌ Invalid grid:**
```vue
<div role="grid">
  <div><!-- ❌ Missing role="row" -->
    <div>Cell</div> <!-- ❌ Missing role="gridcell" -->
  </div>
</div>
```

**✅ Valid grid:**
```vue
<div role="grid" aria-label="Risk heatmap">
  <div role="row">
    <div role="columnheader">Low Impact</div>
    <div role="columnheader">Medium Impact</div>
    <div role="columnheader">High Impact</div>
  </div>
  <div role="row">
    <div role="rowheader">High Likelihood</div>
    <div role="gridcell" tabindex="0" aria-label="High likelihood, low impact: 5 risks">5</div>
    <div role="gridcell" tabindex="0" aria-label="High likelihood, medium impact: 12 risks">12</div>
    <div role="gridcell" tabindex="0" aria-label="High likelihood, high impact: 8 risks">8</div>
  </div>
</div>
```

**Arrow key navigation:**
```vue
<script setup>
const handleGridKeydown = (event, row, col) => {
  const { key } = event;
  let newRow = row;
  let newCol = col;
  
  switch(key) {
    case 'ArrowRight':
      newCol = Math.min(col + 1, maxCols);
      event.preventDefault();
      break;
    case 'ArrowLeft':
      newCol = Math.max(col - 1, 0);
      event.preventDefault();
      break;
    case 'ArrowDown':
      newRow = Math.min(row + 1, maxRows);
      event.preventDefault();
      break;
    case 'ArrowUp':
      newRow = Math.max(row - 1, 0);
      event.preventDefault();
      break;
    case 'Home':
      newCol = 0;
      event.preventDefault();
      break;
    case 'End':
      newCol = maxCols;
      event.preventDefault();
      break;
  }
  
  if (newRow !== row || newCol !== col) {
    focusCell(newRow, newCol);
  }
};
</script>
```

---

## Text Resize

### 200% Zoom Test

Content must reflow properly when zoomed to 200%:

**Common issues:**
- Fixed widths prevent reflow
- Overlapping elements
- Hidden controls
- Horizontal scrolling

**✅ Responsive approach:**
```vue
<style scoped>
/* ❌ Fixed widths break at zoom */
.risk-card {
  width: 300px; /* Breaks at zoom */
}

/* ✅ Use max-width and relative units */
.risk-card {
  max-width: 100%;
  width: clamp(250px, 50%, 600px);
  padding: 1rem;
}

/* ✅ Flex layouts reflow naturally */
.risk-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

/* ✅ Use viewport units carefully */
.chart-container {
  width: 100%;
  max-width: 90vw;
  height: auto;
  aspect-ratio: 16 / 9;
}
</style>
```

**Test at 200% zoom:**
1. Press Ctrl/Cmd + to zoom to 200%
2. Check all content is visible
3. No horizontal scrolling
4. All interactions still work
5. Text doesn't overlap

**Mosaic components handle this well**, but custom layouts need testing.

---

## Multiple Navigation Methods

### Provide Multiple Ways to Find Content

Users should be able to navigate risks in multiple ways:

**1. Direct navigation (URL/bookmarks)**
```vue
<!-- ✅ Meaningful URLs -->
/risks/assessment/123
/risks/by-severity/critical
/risks/by-client/acme-corp
```

**2. Search**
```vue
<div class="search-container">
  <Label for="risk-search">Search Risks</Label>
  <IconField iconPosition="left">
    <InputIcon class="pi pi-search" />
    <InputText 
      id="risk-search"
      v-model="searchQuery"
      placeholder="Search by title, ID, or description"
    />
  </IconField>
</div>
```

**3. Filtering**
```vue
<div class="filters">
  <MultiSelect 
    v-model="selectedSeverities"
    :options="severities"
    placeholder="Filter by Severity"
  />
  <MultiSelect 
    v-model="selectedCategories"
    :options="categories"
    placeholder="Filter by Category"
  />
</div>
```

**4. Breadcrumbs**
```vue
<Breadcrumb :home="home" :model="breadcrumbItems" />

<script setup>
const breadcrumbItems = [
  { label: 'Risks', to: '/risks' },
  { label: 'Assessments', to: '/risks/assessments' },
  { label: 'ACME Corp Q4 2024', to: '/risks/assessments/123' }
];
</script>
```

**5. Table of Contents (for long pages)**
```vue
<nav aria-label="Table of contents">
  <h2>On this page</h2>
  <ul>
    <li><a href="#overview">Overview</a></li>
    <li><a href="#critical-risks">Critical Risks</a></li>
    <li><a href="#controls">Controls</a></li>
    <li><a href="#timeline">Timeline</a></li>
  </ul>
</nav>
```

---

## Testing Checklist

### Manual Testing

1. **Keyboard only:**
   - Tab through all interactive elements
   - Navigate charts with arrow keys
   - Remove filter chips with keyboard
   - Access all table actions
   - Navigate heatmap grid with arrows

2. **Screen reader:**
   - Do chart descriptions make sense?
   - Are data values announced?
   - Can you understand the heatmap structure?
   - Are filter changes announced?
   - Do error messages read clearly?

3. **Zoom to 200%:**
   - Charts still visible and usable
   - Heatmap cells don't overlap
   - Filter chips wrap properly
   - Tables remain readable
   - No horizontal scrolling

4. **Color blindness test:**
   - Use browser DevTools color vision deficiency emulation
   - Check if chart segments are distinguishable
   - Verify heatmap cells are identifiable
   - Ensure status indicators have icons/patterns

### Automated Tools

- **Lighthouse** - Run accessibility audit
- **axe DevTools** - Detailed WCAG checks
- **WAVE** - Visual accessibility feedback

---

## Quick Reference: Data Viz Accessibility

| Element | Must Have | Good Practice |
|---------|-----------|---------------|
| Pie Chart | Text alternative, legend, labels | Data table, patterns, color + text |
| Heatmap | Grid roles, keyboard nav, color + pattern | Tooltips, data table |
| Donut Chart | Text alternative, segment labels | Interactive segments, data table |
| Bar Chart | Axis labels, data labels | Tooltips, keyboard focus |
| Line Chart | Axis labels, legend | Point data on focus |

---

## Common Mistakes to Avoid

1. ❌ Charts with no text alternative
2. ❌ Color-only information in heatmaps
3. ❌ Non-keyboard accessible chart interactions
4. ❌ Generic page titles ("Risks")
5. ❌ Icon buttons without labels
6. ❌ Generic error messages ("Invalid input")
7. ❌ Empty heading elements
8. ❌ Filter chips without keyboard remove
9. ❌ Invalid button markup (button in button)
10. ❌ Grid without proper ARIA roles
11. ❌ Fixed-width layouts that don't reflow
12. ❌ Only one way to navigate content
13. ❌ Focus lost when closing modals
14. ❌ Tab order that jumps around illogically

---

## Resources

- **Chart Accessibility:** https://www.w3.org/WAI/tutorials/images/complex/
- **ARIA Grid Pattern:** https://www.w3.org/WAI/ARIA/apg/patterns/grid/
- **Color Contrast Checker:** https://webaim.org/resources/contrastchecker/
- **Mosaic Component Docs:** Check component-specific accessibility features

---

## Remember

> "If screen reader users can't understand your data visualization, it's not accessible."

Most Risks accessibility issues are fixed by:
1. Providing text alternatives for charts
2. Not relying on color alone
3. Making everything keyboard accessible
4. Using proper ARIA roles for complex widgets
5. Testing with screen readers and at 200% zoom

**When building data viz, always ask: "How would I understand this without seeing it?"**
