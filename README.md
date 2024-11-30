# Clean Architecture & UI Guide für Next.js & TypeScript

## 1. Projekt-Struktur
```
src/
├── app/                 # Next.js App Router
├── components/          # UI Komponenten
│   ├── ui/             # Basis Komponenten
│   └── features/       # Feature Komponenten
├── lib/                # Utilities & Configs
├── types/              # TypeScript Definitionen
├── services/           # External Services
└── utils/              # Helper Funktionen
```

## 2. Komponenten-Architektur
### Server Component Beispiel
```typescript
// app/users/page.tsx
import { getUserList } from '@/services/user-service';

export default async function UsersPage() {
  const users = await getUserList();
  
  return (
    <div className="space-y-4">
      <h1 className="text-2xl font-bold">Benutzer</h1>
      <UserList users={users} />
    </div>
  );
}
```

### Client Component Beispiel
```typescript
// components/features/user-list.tsx
'use client';

interface User {
  id: string;
  name: string;
  email: string;
}

interface Props {
  users: User[];
}

export function UserList({ users }: Props) {
  const [selectedId, setSelectedId] = useState<string | null>(null);
  
  return (
    <div className="grid gap-4">
      {users.map(user => (
        <UserCard 
          key={user.id}
          user={user}
          isSelected={user.id === selectedId}
          onSelect={() => setSelectedId(user.id)}
        />
      ))}
    </div>
  );
}
```

## 3. API Integration
### Service Layer
```typescript
// services/user-service.ts
export class UserService {
  private readonly baseUrl = process.env.NEXT_PUBLIC_API_URL;

  async getUserList(): Promise<User[]> {
    try {
      const res = await fetch(`${this.baseUrl}/users`);
      if (!res.ok) throw new Error('Fehler beim Laden der Benutzer');
      return res.json();
    } catch (error) {
      handleError(error);
      return [];
    }
  }
}
```

### API Route Handler
```typescript
// app/api/users/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  try {
    const users = await prisma.user.findMany();
    return NextResponse.json(users);
  } catch (error) {
    return NextResponse.json(
      { error: 'Fehler beim Laden der Daten' },
      { status: 500 }
    );
  }
}
```

## 4. State Management
### Zustand Store
```typescript
// lib/store.ts
import create from 'zustand';

interface UserStore {
  users: User[];
  selectedUser: User | null;
  setUsers: (users: User[]) => void;
  selectUser: (user: User) => void;
}

export const useUserStore = create<UserStore>((set) => ({
  users: [],
  selectedUser: null,
  setUsers: (users) => set({ users }),
  selectUser: (user) => set({ selectedUser: user }),
}));
```

## 5. Typen-Definitionen
```typescript
// types/user.ts
export interface User {
  id: string;
  name: string;
  email: string;
  role: UserRole;
  createdAt: Date;
}

export enum UserRole {
  ADMIN = 'ADMIN',
  USER = 'USER',
}

export type UserCreateInput = Omit<User, 'id' | 'createdAt'>;
```

## 6. Error Handling
```typescript
// lib/error-handling.ts
export class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public status: number
  ) {
    super(message);
  }
}

export function handleError(error: unknown) {
  if (error instanceof AppError) {
    // Handle known errors
    logger.error({
      code: error.code,
      message: error.message
    });
  } else {
    // Handle unknown errors
    logger.error('Unbekannter Fehler:', error);
  }
}
```

## 7. Performance Optimierung
### Image Optimization
```typescript
// components/ui/optimized-image.tsx
import Image from 'next/image';

interface Props {
  src: string;
  alt: string;
  width?: number;
  height?: number;
}

export function OptimizedImage({ src, alt, width = 300, height = 200 }: Props) {
  return (
    <div className="relative aspect-video">
      <Image
        src={src}
        alt={alt}
        fill
        sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
        className="object-cover"
        loading="lazy"
      />
    </div>
  );
}
```

## 8. Best Practices Checkliste
### Entwicklung
- [ ] Server vs. Client Components korrekt eingesetzt
- [ ] TypeScript strict mode aktiviert
- [ ] Keine any Types
- [ ] Error Boundaries implementiert
- [ ] Performance Monitoring eingerichtet

### Code Qualität
- [ ] ESLint Regeln definiert
- [ ] Prettier konfiguriert
- [ ] Unit Tests für kritische Funktionen
- [ ] E2E Tests für wichtige Flows
- [ ] Dokumentation aktuell

## Implementierungs-Schritte:
1. Projekt-Struktur aufsetzen
2. Basis-Komponenten entwickeln
3. Features implementieren
4. Tests schreiben
5. Performance optimieren
6. Monitoring einrichten

## Tipps für den Produktivbetrieb:
- Vercel Analytics nutzen
- Error Tracking einrichten
- Performance Monitoring aktivieren
- Regular Security Audits durchführen
- Dependency Updates automatisieren
