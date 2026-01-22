# Before/After Examples

Real-world simplification examples.

## Example 1: Data Processing

### Before (42 lines)
```typescript
async function processUserData(userId: string) {
  try {
    const user = await db.users.findOne({ id: userId })
    if (user) {
      if (user.status === 'active') {
        const orders = await db.orders.find({ userId: user.id })
        if (orders && orders.length > 0) {
          const total = orders.reduce((sum, order) => {
            return sum + order.amount
          }, 0)
          return {
            user: user,
            orderCount: orders.length,
            totalSpent: total
          }
        } else {
          return {
            user: user,
            orderCount: 0,
            totalSpent: 0
          }
        }
      } else {
        return null
      }
    } else {
      return null
    }
  } catch (error) {
    console.error(error)
    return null
  }
}
```

### After (18 lines)
```typescript
async function processUserData(userId: string) {
  const user = await db.users.findOne({ id: userId })
  if (!user || user.status !== 'active') return null

  const orders = await db.orders.find({ userId: user.id })
  const total = orders.reduce((sum, order) => sum + order.amount, 0)

  return {
    user,
    orderCount: orders.length,
    totalSpent: total
  }
}
```

**Changes**: Early returns, removed try/catch (let errors propagate), object shorthand, inline reduce.

---

## Example 2: React Component

### Before (35 lines)
```tsx
function UserCard(props: { user: User; onEdit: () => void; onDelete: () => void }) {
  const handleEdit = () => {
    props.onEdit()
  }

  const handleDelete = () => {
    props.onDelete()
  }

  return (
    <div className="card">
      <div className="card-header">
        <h3>{props.user.name}</h3>
      </div>
      <div className="card-body">
        <p>{props.user.email}</p>
        <p>{props.user.role}</p>
      </div>
      <div className="card-footer">
        <button onClick={handleEdit}>Edit</button>
        <button onClick={handleDelete}>Delete</button>
      </div>
    </div>
  )
}
```

### After (20 lines)
```tsx
function UserCard({ user, onEdit, onDelete }: Props) {
  const { name, email, role } = user

  return (
    <div className="card">
      <div className="card-header">
        <h3>{name}</h3>
      </div>
      <div className="card-body">
        <p>{email}</p>
        <p>{role}</p>
      </div>
      <div className="card-footer">
        <button onClick={onEdit}>Edit</button>
        <button onClick={onDelete}>Delete</button>
      </div>
    </div>
  )
}
```

**Changes**: Destructured props, removed wrapper functions, extracted user properties.

---

## Example 3: Conditional Logic

### Before
```typescript
function getDiscount(user: User, cart: Cart): number {
  let discount = 0

  if (user.isPremium === true) {
    if (cart.total >= 100) {
      discount = 0.2
    } else if (cart.total >= 50) {
      discount = 0.1
    } else {
      discount = 0.05
    }
  } else {
    if (cart.total >= 100) {
      discount = 0.1
    } else if (cart.total >= 50) {
      discount = 0.05
    }
  }

  return discount
}
```

### After
```typescript
function getDiscount(user: User, cart: Cart): number {
  const { total } = cart
  const base = user.isPremium ? 0.05 : 0

  if (total >= 100) return base + 0.1
  if (total >= 50) return base + 0.05
  return base
}
```

**Changes**: Extracted pattern, early returns, removed redundant `=== true`.
