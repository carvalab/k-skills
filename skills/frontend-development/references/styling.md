# Styling Guide

MUI styling with sx prop and theme integration.

## Inline vs Separate

**< 100 lines**: Inline in component
**> 100 lines**: Separate `.styles.ts` file

```typescript
// Inline (< 100 lines)
import type { SxProps, Theme } from '@mui/material';

const styles: Record<string, SxProps<Theme>> = {
    container: { p: 2, display: 'flex', flexDirection: 'column' },
    header: { mb: 2, borderBottom: '1px solid', borderColor: 'divider' },
};

export function MyComponent() {
    return (
        <Box sx={styles.container}>
            <Box sx={styles.header}>Title</Box>
        </Box>
    );
}
```

## sx Prop Patterns

### Basic
```typescript
<Box sx={{ p: 2, mb: 3, display: 'flex' }}>Content</Box>
```

### Theme Access
```typescript
<Box
    sx={{
        backgroundColor: (theme) => theme.palette.primary.main,
        borderRadius: (theme) => theme.shape.borderRadius,
    }}
/>
```

### Responsive
```typescript
<Box
    sx={{
        p: { xs: 1, sm: 2, md: 3 },
        width: { xs: '100%', md: '50%' },
    }}
/>
```

### Pseudo-Selectors
```typescript
<Box
    sx={{
        '&:hover': { backgroundColor: 'rgba(0,0,0,0.05)' },
        '& .child': { color: 'primary.main' },
    }}
/>
```

## MUI Grid

Use `size` prop:

```typescript
import { Grid } from '@mui/material';

// ✅ Current syntax
<Grid container spacing={2}>
    <Grid size={{ xs: 12, md: 6 }}>Left</Grid>
    <Grid size={{ xs: 12, md: 6 }}>Right</Grid>
</Grid>

// ❌ Old syntax
<Grid xs={12} md={6}>
```

### Nested
```typescript
<Grid container spacing={2}>
    <Grid size={{ xs: 12, md: 8 }}>
        <Grid container spacing={1}>
            <Grid size={{ xs: 6 }}>Nested 1</Grid>
            <Grid size={{ xs: 6 }}>Nested 2</Grid>
        </Grid>
    </Grid>
</Grid>
```

## Avoid

```typescript
// ❌ makeStyles (deprecated)
import { makeStyles } from '@mui/styles';

// ❌ styled() - sx is more flexible
import { styled } from '@mui/material/styles';
```

**Use sx prop instead.**

## Spacing

| Value | Pixels |
|-------|--------|
| 0.5 | 4px |
| 1 | 8px |
| 2 | 16px |
| 3 | 24px |
| 4 | 32px |

```typescript
p: 2           // All: 16px
px: 2          // Horizontal
py: 2          // Vertical
pt: 2          // Top only
```

## Common Patterns

### Flexbox
```typescript
const styles = {
    row: { display: 'flex', alignItems: 'center', gap: 2 },
    column: { display: 'flex', flexDirection: 'column', gap: 1 },
    between: { display: 'flex', justifyContent: 'space-between' },
};
```

### Positioning
```typescript
const styles = {
    sticky: { position: 'sticky', top: 0, zIndex: 1000 },
    absolute: { position: 'absolute', top: 0, right: 0 },
};
```

### Card
```typescript
<Paper sx={{ p: 3, borderRadius: 2, '&:hover': { boxShadow: 4 } }}>
    Content
</Paper>
```

## Color Shortcuts

```typescript
<Box sx={{ color: 'primary.main' }} />
<Box sx={{ bgcolor: 'secondary.light' }} />
<Typography color='text.secondary' />
```

## Conditional Styles

```typescript
<Box
    sx={{
        ...(isActive && {
            backgroundColor: 'primary.light',
            border: '2px solid',
            borderColor: 'primary.main',
        }),
    }}
/>
```

## Code Style

- 4 spaces indentation
- Single quotes
- Trailing commas
