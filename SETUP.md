# Class Connect - Supabase Setup Guide

This guide will help you set up the Supabase backend for Class Connect.

## Prerequisites

1. A Supabase account (sign up at https://supabase.com)
2. A Supabase project created

## Step 1: Get Your Supabase Credentials

1. Go to your Supabase project dashboard
2. Click on **Settings** → **API**
3. Copy your **Project URL** and **anon/public key**
4. Open `config.js` in your project
5. Replace the placeholders:
   ```javascript
   const SUPABASE_URL = 'YOUR_SUPABASE_URL';  // Replace with your Project URL
   const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';  // Replace with your anon key
   ```

## Step 2: Create Database Tables

Go to **SQL Editor** in your Supabase dashboard and run the following SQL commands:

### 1. Create `users` table

```sql
CREATE TABLE users (
  id UUID REFERENCES auth.users(id) PRIMARY KEY,
  name TEXT NOT NULL,
  email TEXT NOT NULL,
  class TEXT NOT NULL,
  profile_pic TEXT,
  bio TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Policy: Users can read all profiles
CREATE POLICY "Users can read all profiles" ON users
  FOR SELECT USING (true);

-- Policy: Users can update their own profile
CREATE POLICY "Users can update own profile" ON users
  FOR UPDATE USING (auth.uid() = id);

-- Policy: Users can insert their own profile
CREATE POLICY "Users can insert own profile" ON users
  FOR INSERT WITH CHECK (auth.uid() = id);
```

### 2. Create `posts` table

```sql
CREATE TABLE posts (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) NOT NULL,
  content TEXT NOT NULL,
  image_url TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

-- Policy: Authenticated users can read all posts
CREATE POLICY "Authenticated users can read posts" ON posts
  FOR SELECT USING (auth.role() = 'authenticated');

-- Policy: Users can create their own posts
CREATE POLICY "Users can create posts" ON posts
  FOR INSERT WITH CHECK (auth.uid() = user_id);

-- Policy: Users can delete their own posts
CREATE POLICY "Users can delete own posts" ON posts
  FOR DELETE USING (auth.uid() = user_id);
```

### 3. Create `events` table

```sql
CREATE TABLE events (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  title TEXT NOT NULL,
  date DATE NOT NULL,
  description TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE events ENABLE ROW LEVEL SECURITY;

-- Policy: Authenticated users can read all events
CREATE POLICY "Authenticated users can read events" ON events
  FOR SELECT USING (auth.role() = 'authenticated');

-- Policy: Only authenticated users can create events (you can restrict this further)
CREATE POLICY "Authenticated users can create events" ON events
  FOR INSERT WITH CHECK (auth.role() = 'authenticated');
```

### 4. Create `friends` table (for future use)

```sql
CREATE TABLE friends (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) NOT NULL,
  friend_id UUID REFERENCES auth.users(id) NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(user_id, friend_id)
);

-- Enable Row Level Security
ALTER TABLE friends ENABLE ROW LEVEL SECURITY;

-- Policy: Users can read their own friendships
CREATE POLICY "Users can read own friendships" ON friends
  FOR SELECT USING (auth.uid() = user_id OR auth.uid() = friend_id);

-- Policy: Users can create their own friendships
CREATE POLICY "Users can create friendships" ON friends
  FOR INSERT WITH CHECK (auth.uid() = user_id);
```

### 5. Create `messages` table (for chat)

```sql
CREATE TABLE messages (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) NOT NULL,
  text TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;

-- Policy: Authenticated users can read all messages
CREATE POLICY "Authenticated users can read messages" ON messages
  FOR SELECT USING (auth.role() = 'authenticated');

-- Policy: Users can create messages
CREATE POLICY "Users can create messages" ON messages
  FOR INSERT WITH CHECK (auth.uid() = user_id);
```

## Step 3: Enable Realtime for Messages

1. Go to **Database** → **Replication** in your Supabase dashboard
2. Find the `messages` table
3. Toggle **Enable Realtime** to ON

This allows real-time chat functionality.

## Step 4: Create Storage Bucket for Gallery

1. Go to **Storage** in your Supabase dashboard
2. Click **New bucket**
3. Name it: `gallery`
4. Make it **Public** (so images can be viewed)
5. Click **Create bucket**

### Set up Storage Policies

Go to **Storage** → **Policies** for the `gallery` bucket and create:

```sql
-- Policy: Anyone can view gallery images
CREATE POLICY "Public Access" ON storage.objects
  FOR SELECT USING (bucket_id = 'gallery');

-- Policy: Authenticated users can upload to gallery
CREATE POLICY "Authenticated users can upload" ON storage.objects
  FOR INSERT WITH CHECK (bucket_id = 'gallery' AND auth.role() = 'authenticated');
```

## Step 5: Configure Authentication

1. Go to **Authentication** → **Settings** in your Supabase dashboard
2. Under **Site URL**, add your website URL (or `http://localhost` for local development)
3. Under **Redirect URLs**, add your website URLs

## Step 6: Test Your Setup

1. Open `login.html` in your browser
2. Create a new account using the signup page
3. After signing up, you should be redirected to the home page
4. Try creating a post, viewing your profile, etc.

## Troubleshooting

### Error: "relation does not exist"
- Make sure you've run all the SQL commands in Step 2
- Check that table names match exactly (case-sensitive)

### Error: "new row violates row-level security policy"
- Check your RLS policies are set up correctly
- Make sure the user is authenticated

### Gallery not loading images
- Ensure the `gallery` bucket exists and is public
- Check storage policies are set correctly

### Chat not working in real-time
- Make sure Realtime is enabled for the `messages` table
- Check your Supabase project has Realtime enabled

### Authentication not working
- Verify your `SUPABASE_URL` and `SUPABASE_ANON_KEY` in `config.js`
- Check that authentication is enabled in your Supabase project

## Next Steps

1. Add sample data to test the application:
   - Insert some events in the `events` table
   - Upload some images to the `gallery` bucket
   
2. Customize the application:
   - Modify colors in `style.css`
   - Add more features as needed

3. Deploy your application:
   - Use Netlify, Vercel, or GitHub Pages
   - Update your Supabase redirect URLs to match your deployed URL

## Database Schema Summary

- **users**: User profiles (id, name, email, class, profile_pic, bio)
- **posts**: User posts (id, user_id, content, image_url, created_at)
- **events**: School events (id, title, date, description)
- **friends**: Friend relationships (id, user_id, friend_id)
- **messages**: Chat messages (id, user_id, text, created_at)

## Security Notes

- All tables have Row Level Security (RLS) enabled
- Users can only modify their own data
- Gallery bucket is public for viewing but requires authentication for uploads
- Adjust policies as needed for your specific requirements

---

**Created by:** Adiptha Wekadapala  
**For:** Kegalu Vidyalaya - Class M6

