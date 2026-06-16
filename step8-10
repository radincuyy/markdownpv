# Sociyo — Handoff Plan Step 8, 9, 10

> **Untuk:** Teman developer Radin
> **Base branch:** `develop`
> **Commit terakhir:** `720b7f5` — Step 7 selesai

---

## ⚡ Git Workflow (BACA DULU!)

### 1. Clone & Buat Branch Baru

```bash
git clone <repo-url>
cd AnimaVibeSocial
git checkout develop
git pull origin develop
git checkout -b feature/step-8-10-story-profile-cleanup
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Pastikan `.env` Sudah Ada

Minta file `.env` dari Radin — isinya Firebase credentials. Taruh di root project.

### 4. Kerja & Commit Per Step

```bash
# Setelah selesai satu step, commit:
git add -A
git commit -m "feat: story service layer Firestore"

# Lanjut step berikutnya...
git add -A
git commit -m "feat: story viewer screen dengan animasi"
```

### 5. Push & Buat Pull Request

```bash
git push origin feature/step-8-10-story-profile-cleanup
```

Lalu buat **Pull Request** di GitHub:
- **Base:** `develop`
- **Compare:** `feature/step-8-10-story-profile-cleanup`
- **Title:** `feat: Step 8-10 — Story Viewer, Profile Grid, Cleanup`
- **Description:** Ringkas apa yang sudah dikerjakan per step

> [!IMPORTANT]
> **JANGAN merge sendiri.** Tunggu Radin review dan approve dulu.

### 6. Sebelum TypeScript Check Setiap Commit

```bash
npx tsc --noEmit
```

Harus **0 error** setiap kali commit.

## Konteks Penting

### Stack & Constraint

- **Framework:** Expo (React Native) + TypeScript
- **State:** Zustand (store pattern di `src/store/`)
- **Backend:** Firebase Firestore (database `new-db`) + Storage
- **Animasi:** Wajib pakai **Reanimated 2** + **Gesture Handler** (JANGAN pakai `Animated` API bawaan RN)
- **Styling:** Dark/Light mode via `useThemeStore` + `colors[mode]` palette
- **Image:** Pakai `expo-image` (`Image` dari `expo-image`, bukan dari `react-native`)
- **Navigasi:** React Navigation (Stack + Tab + Drawer)

### Pattern yang Sudah Ada (Ikuti!)

| Layer | Contoh file | Pattern |
|-------|-------------|---------|
| Service | [postService.ts](file:///g:/Sociyo/AnimaVibeSocial/src/services/postService.ts) | Firestore CRUD, helper `str()` `num()`, timestamp conversion |
| Store | [usePostStore.ts](file:///g:/Sociyo/AnimaVibeSocial/src/store/usePostStore.ts) | Zustand `create()`, async actions, optimistic updates |
| Screen | [PostDetailScreen.tsx](file:///g:/Sociyo/AnimaVibeSocial/src/screens/feed/PostDetailScreen.tsx) | `Screen` wrapper, `palette` dari `colors[mode]`, memoized components |
| Component | [AnimatedPostCard.tsx](file:///g:/Sociyo/AnimaVibeSocial/src/components/AnimatedPostCard.tsx) | Reanimated `useSharedValue` + `useAnimatedStyle`, gesture handler |
| Types | [social.ts](file:///g:/Sociyo/AnimaVibeSocial/src/types/social.ts) | Type-safe Firestore docs (`PostDoc`, `CommentDoc`, dll) |
| Nav types | [navigation.ts](file:///g:/Sociyo/AnimaVibeSocial/src/types/navigation.ts) | `RootStackParamList` untuk Stack screens |

### Firebase Config

```typescript
// src/services/firebase.ts
export const firestore = getFirestore(firebaseApp, 'new-db');
export const firebaseStorage = getStorage(firebaseApp);
```

---

## Step 8: Immersive Story Viewer

### 8A. Firestore Schema

Collection: `stories`

```
stories/{storyId}
├── authorId: string
├── imageUrl: string          // wajib ada gambar
├── caption: string | null
├── createdAt: Timestamp
├── expiresAt: Timestamp      // 24 jam dari createdAt
├── viewedBy: string[]        // array of user IDs yang sudah lihat
```

### 8B. File Baru: `src/services/storyService.ts`

Ikuti pattern [postService.ts](file:///g:/Sociyo/AnimaVibeSocial/src/services/postService.ts).

**Fungsi yang perlu dibuat:**

```typescript
// Upload story (gambar wajib)
createStory(authorId: string, imageUri: string, caption?: string): Promise<string>

// Ambil semua story yang belum expired, grouped by user
getStoryGroups(): Promise<StoryGroup[]>

// Tandai story sebagai sudah dilihat
markStoryViewed(storyId: string, userId: string): Promise<void>
```

**Catatan penting:**
- Filter `expiresAt > now` untuk hanya ambil story aktif
- Group by `authorId`, sort by `createdAt` ascending per group
- Cek `viewedBy.includes(currentUserId)` untuk set `hasUnviewed`
- Upload gambar ke Storage path: `stories/{authorId}/{timestamp}.jpg`

### 8C. File Baru: `src/store/useStoryStore.ts`

Ikuti pattern [usePostStore.ts](file:///g:/Sociyo/AnimaVibeSocial/src/store/usePostStore.ts).

```typescript
type StoryState = {
  groups: StoryGroup[];        // dari social.ts, sudah ada type-nya
  loading: boolean;
  fetchStories: () => Promise<void>;
  markViewed: (storyId: string) => Promise<void>;
}
```

### 8D. Rewrite: `src/screens/story/StoryViewerScreen.tsx`

File saat ini: [StoryViewerScreen.tsx](file:///g:/Sociyo/AnimaVibeSocial/src/screens/story/StoryViewerScreen.tsx) — hanya placeholder kosong.

**Fitur wajib (dari memory.md):**

1. **Progress bar animasi** di atas — horizontal bars per story, auto-advance tiap 5 detik
   - Pakai `useSharedValue` + `withTiming` durasi 5000ms
   - Bar yang aktif animasi dari 0% ke 100%
   
2. **Tap kanan/kiri** — next/prev story
   - Tap area kanan (>50% width) = next
   - Tap area kiri (<50% width) = prev
   
3. **Swipe horizontal** — next/prev user group
   - Pakai `Gesture.Pan()` dengan threshold
   - `runOnJS()` untuk update state

4. **Pause/Resume** — long press pause, release resume
   - `Gesture.LongPress()` → pause `cancelAnimation()` + simpan progress
   - Release → resume `withTiming()` dari progress sisa

5. **Fullscreen dark background** — `Screen` dengan `padded={false}`, background hitam

**Layout:**
```
┌────────────────────────────┐
│ ▓▓▓▓▓▓ ░░░░░░ ░░░░░░ ░░░░ │  ← progress bars
│ [avatar] username    [X]   │  ← header
│                            │
│                            │
│        STORY IMAGE         │  ← full screen Image
│                            │
│                            │
│   [reply input field]      │  ← optional reply
└────────────────────────────┘
```

**Navigation params** sudah ada di [navigation.ts](file:///g:/Sociyo/AnimaVibeSocial/src/types/navigation.ts):
```typescript
StoryViewer: { storyId?: string }
```

> [!IMPORTANT]
> Ubah param jadi `StoryViewer: { userId: string }` agar bisa load semua story dari user tersebut. Update juga di [AppNavigator.tsx](file:///g:/Sociyo/AnimaVibeSocial/src/navigation/AppNavigator.tsx).

### 8E. Story Ring di FeedScreen

Tambahkan horizontal `FlatList` di atas feed posts di [FeedScreen.tsx](file:///g:/Sociyo/AnimaVibeSocial/src/screens/feed/FeedScreen.tsx):

- Render circle avatars dengan gradient ring (Reanimated rotating gradient)
- Ring berputar kalau `hasUnviewed === true`, ring biasa kalau sudah dilihat
- Tap → navigate ke `StoryViewer` screen
- Item pertama = "+" untuk create story sendiri

---

## Step 9: Profile Posts Grid

### 9A. Update: `src/screens/profile/ProfileScreen.tsx`

File saat ini: [ProfileScreen.tsx](file:///g:/Sociyo/AnimaVibeSocial/src/screens/profile/ProfileScreen.tsx) — ada section "Recent posts" tapi isinya `EmptyState`.

**Yang perlu dilakukan:**

1. **Fetch posts milik user** — tambah fungsi di [postService.ts](file:///g:/Sociyo/AnimaVibeSocial/src/services/postService.ts):
   ```typescript
   getUserPosts(userId: string, maxResults?: number): Promise<Post[]>
   ```
   Query: `where('authorId', '==', userId)`, `orderBy('createdAt', 'desc')`

2. **Grid 3×3** — sama seperti di [SearchScreen.tsx](file:///g:/Sociyo/AnimaVibeSocial/src/screens/search/SearchScreen.tsx):
   - Pakai `FlatList` dengan `numColumns={3}`
   - Tile square, tap → navigate ke `PostDetail`
   - Post tanpa gambar tetap tampil tapi pakai fallback (warna solid + caption preview)

3. **Ganti `EmptyState`** — hapus EmptyState, ganti dengan grid
   - Kalau belum ada post, tampilkan empty state
   - Kalau ada post, tampilkan grid

**Layout setelah update:**
```
┌──────────────────────────┐
│ [Avatar]  Nama            │
│           @username   [✏]│
│                          │
│  [Posts] [Followers] [Following] │
│                          │
│  Bio section             │
│                          │
│  ─── Posts ──────────── │
│  ┌────┐ ┌────┐ ┌────┐  │
│  │    │ │    │ │    │  │
│  └────┘ └────┘ └────┘  │
│  ┌────┐ ┌────┐ ┌────┐  │
│  │    │ │    │ │    │  │
│  └────┘ └────┘ └────┘  │
└──────────────────────────┘
```

> [!NOTE]
> Grid tile size: `(screenWidth - padding) / 3` dengan gap 2px, exact same pattern dari SearchScreen.

---

## Step 10: Final Type Cleanup

### 10A. Update: `src/types/social.ts`

1. **Tambah `StoryDoc`** type untuk Firestore document:
   ```typescript
   export type StoryDoc = {
     authorId: string;
     imageUrl: string;
     caption: string | null;
     createdAt: unknown;
     expiresAt: unknown;
     viewedBy: string[];
   };
   ```

2. **Review semua type** — pastikan field yang optional pakai `?` consistent
3. **Hapus field yang tidak dipakai** kalau ada

### 10B. Verifikasi TypeScript

```bash
npx tsc --noEmit
```

Harus **0 error**. Jangan pakai `any` atau `@ts-ignore`.

### 10C. Cleanup Checklist

- [ ] Semua `console.log` debug dihapus
- [ ] Semua komentar `// TODO` diselesaikan atau dihapus
- [ ] Semua import yang tidak dipakai dihapus
- [ ] Test manual: Login → Feed → Create Post → PostDetail → Search → Profile → Story → Logout

---

## Urutan Kerja yang Disarankan

```mermaid
graph TD
    A["8B: storyService.ts"] --> B["8C: useStoryStore.ts"]
    B --> C["8D: StoryViewerScreen.tsx"]
    C --> D["8E: Story ring di FeedScreen"]
    D --> E["9A: getUserPosts di postService"]
    E --> F["9A: Profile grid di ProfileScreen"]
    F --> G["10: Type cleanup + verifikasi"]
```

**Commit setiap step**, contoh:
```
feat: story service layer Firestore
feat: story store Zustand
feat: story viewer screen dengan animasi
feat: story ring di feed screen
feat: profile posts grid 3x3
chore: type cleanup dan verifikasi final
```

---

## File yang TIDAK BOLEH Diubah

> [!CAUTION]
> File berikut sudah stabil dan tested. Jangan ubah kecuali benar-benar perlu:
> - `src/services/firebase.ts` — config Firebase
> - `src/store/useAuthStore.ts` — auth flow
> - `src/theme/colors.ts` — color palette
> - `src/components/Screen.tsx` — screen wrapper

## File yang Boleh Dimodifikasi

| File | Alasan |
|------|--------|
| `src/services/postService.ts` | Tambah `getUserPosts()` |
| `src/types/social.ts` | Tambah `StoryDoc` |
| `src/types/navigation.ts` | Update `StoryViewer` params |
| `src/navigation/AppNavigator.tsx` | Update StoryViewer route |
| `src/screens/feed/FeedScreen.tsx` | Tambah story ring bar |
| `src/screens/story/StoryViewerScreen.tsx` | Full rewrite |
| `src/screens/profile/ProfileScreen.tsx` | Tambah posts grid |

## File Baru

| File | Deskripsi |
|------|-----------|
| `src/services/storyService.ts` | CRUD story + mark viewed |
| `src/store/useStoryStore.ts` | Zustand store untuk story groups |
