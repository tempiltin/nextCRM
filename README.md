# nextCRM
# Next.js asosida CRM tizimi qurish yo‘riqnomasi

## 1. Loyihani yaratish va konfiguratsiya qilish

CRM tizimini Next.js asosida yaratish uchun quyidagi bosqichlarni bajaramiz:

### 1.1. Next.js Loyihasini Yaratish
```sh
npx create-next-app@latest crm-frontend
cd crm-frontend
```
- TypeScript tanlash kerak.
- **`app/` router strukturasidan** foydalanish lozim.

### 1.2. Muhim kutubxonalarni o‘rnatish
```sh
npm install axios zustand tailwindcss @shadcn/ui @next/font next-auth
npx tailwindcss init -p
```

**Tailwind konfiguratsiyasi (`tailwind.config.js`)**
```js
module.exports = {
  content: ["./app/**/*.{js,ts,jsx,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

---

## 2. CRM uchun papkalar tuzilmasi

```
/crm-frontend
 ├── /app
 │   ├── /dashboard
 │   │   ├── page.tsx         --> Dashboard sahifasi
 │   │   ├── layout.tsx       --> Sidebar va navbar
 │   │   ├── loading.tsx      --> Yuklanish sahifasi
 │   │   ├── error.tsx        --> Xatoliklar boshqaruvi
 │   ├── /clients
 │   │   ├── page.tsx         --> Mijozlar ro‘yxati
 │   │   ├── [id]/page.tsx    --> Mijozning individual sahifasi
 │   ├── /auth
 │   │   ├── login.tsx        --> Kirish sahifasi
 │   │   ├── register.tsx     --> Ro‘yxatdan o‘tish sahifasi
 │   ├── layout.tsx           --> CRM asosiy layouti
 │   ├── page.tsx             --> Bosh sahifa
 ├── /components              --> UI komponentlar
 ├── /lib                     --> API chaqiruvlar va config
 ├── /middleware.ts           --> Auth va xavfsizlik
 ├── /public                  --> Statik fayllar
 ├── /styles                  --> CSS yoki Tailwind
 ├── /utils                    --> Yordamchi funksiyalar
 ├── next.config.js           --> Next.js konfiguratsiyasi
 ├── .env.local               --> Muhim API kalitlari
```

---

## 3. UI/UX va Dizayn Tavsiyalari

### 3.1. UI Kutubxonasi
- **ShadCN/UI**: Osonlashtirilgan UI komponentlar uchun.
- **Tailwind CSS**: Moslashuvchan va tezkor UI uchun.
- **Dark Mode**: CRM uchun majburiy bo‘lishi mumkin.
- **Reusability**: Har bir UI komponenti `components/` ichida modullar sifatida bo‘lishi kerak.

### 3.2. Asosiy UI Komponentlar
- **Sidebar** (`components/Sidebar.tsx`)
- **Navbar** (`components/Navbar.tsx`)
- **Modal** (`components/Modal.tsx`)
- **Formlar** (`components/Form.tsx`)

---

## 4. API Konfiguratsiyasi va Ma’lumotlarni Olish

### 4.1. API konfiguratsiyasi (`lib/api.ts`)
```ts
const BASE_URL = process.env.NEXT_PUBLIC_API_URL || "https://crm-backend.com/api";

export const getClients = async () => {
  const res = await fetch(`${BASE_URL}/clients`, { cache: "no-store" });
  return res.json();
};
```

### 4.2. API orqali mijozlarni olish (`app/clients/page.tsx`)
```tsx
import { getClients } from "@/lib/api";

export default async function ClientsPage() {
  const clients = await getClients();

  return (
    <div>
      <h1>Mijozlar ro‘yxati</h1>
      {clients.map(client => (
        <p key={client.id}>{client.name}</p>
      ))}
    </div>
  );
}
```

---

## 5. Xavfsizlik Choralari

### 5.1. Middleware orqali himoya qilish (`middleware.ts`)
```ts
import { NextRequest, NextResponse } from "next/server";

export function middleware(req: NextRequest) {
  const token = req.cookies.get("auth-token");

  if (!token && req.nextUrl.pathname !== "/auth/login") {
    return NextResponse.redirect(new URL("/auth/login", req.url));
  }
  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*", "/clients/:path*"],
};
```

### 5.2. NextAuth bilan autentifikatsiya qilish (`auth.ts`)
```ts
import NextAuth from "next-auth";
import CredentialsProvider from "next-auth/providers/credentials";

export default NextAuth({
  providers: [
    CredentialsProvider({
      name: "Credentials",
      credentials: {
        username: { label: "Username", type: "text" },
        password: { label: "Password", type: "password" },
      },
      async authorize(credentials) {
        const res = await fetch("https://crm-backend.com/api/login", {
          method: "POST",
          body: JSON.stringify(credentials),
          headers: { "Content-Type": "application/json" },
        });
        const user = await res.json();
        return user || null;
      },
    }),
  ],
  session: { strategy: "jwt" },
});
```

---

## 6. Xatoliklarni boshqarish

### 6.1. Global Xatolik Sahifasi (`app/error.tsx`)
```tsx
"use client";

export default function Error({ error }: { error: Error }) {
  return (
    <div className="text-red-600">
      <h1>Xatolik yuz berdi</h1>
      <p>{error.message}</p>
    </div>
  );
}
```

---

## 7. Deployment va Natijaviy Tavsiyalar

### 7.1. CRM'ni Vercel yoki Docker orqali Deploy qilish
- **Vercel uchun:**
  ```sh
  vercel deploy
  ```
- **Docker uchun:**
  ```dockerfile
  FROM node:18-alpine
  WORKDIR /app
  COPY . .
  RUN npm install
  RUN npm run build
  CMD ["npm", "start"]
  ```

### 7.2. CRM uchun Security Tavsiyalar
✅ **Xavfsiz Cookie-lar ishlatish** (`httpOnly`, `Secure`, `SameSite`)  
✅ **Rate Limiting qo‘shish** (`express-rate-limit` yoki Next.js API middleware)  
✅ **Helmet bilan qo‘shimcha xavfsizlik** (`helmet` kutubxonasidan foydalanish)  

---

Bu yo‘riqnoma Next.js asosida yirik CRM tizimini tuzish uchun zarur barcha muhim jihatlarni qamrab oladi.

