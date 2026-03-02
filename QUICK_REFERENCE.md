# Quick Reference - Google OAuth Fix

## ✅ What Was Fixed

### Backend Changes (app.py)

**1. Session Configuration (Line 14-26)**
```python
# Changed from:
app.secret_key = os.environ.get("SECRET_KEY", "dev_secret")
CORS(app)

# To:
app.secret_key = os.environ.get("SECRET_KEY", "dev_secret")
app.config.update(
    SESSION_COOKIE_SECURE=True,
    SESSION_COOKIE_HTTPONLY=True,
    SESSION_COOKIE_SAMESITE="Lax",
    PERMANENT_SESSION_LIFETIME=86400
)
CORS(app, supports_credentials=True)
```

**2. Google Callback Route (Line 832-891)**
- Now saves user info in Flask session
- Creates/updates user in database
- Returns redirect with proper authentication

**3. New Endpoints Added:**
- `GET /api/auth/user` - Check logged-in user from session
- `POST /api/auth/logout` - Clear session and logout

---

### Frontend Changes (index.html)

**1. Dynamic BASE_URL (Line 648)**
```javascript
// Changed from:
const BASE_URL = 'http://127.0.0.1:5000';

// To:
const BASE_URL = window.location.origin !== 'null' && window.location.origin !== '' 
    ? window.location.origin
    : 'http://127.0.0.1:5000';
```

**2. Auth Check (Line 720-765)**
- Now checks `/api/auth/user` FIRST
- Uses `credentials: 'include'` for cookies
- Falls back to localStorage if no session

**3. Login/Signup Functions**
- Added `credentials: 'include'` to all requests

**4. Logout Function (Line 818-834)**
- Now calls backend `/api/auth/logout`
- Clears both server session and localStorage

---

## 🚀 Deployment Steps

### 1. Generate Secret Key
```python
import secrets
print(secrets.token_hex(32))
```

### 2. Set Render Environment Variables
In Render Dashboard > Environment:
```
SECRET_KEY=<generated-key>
GOOGLE_CLIENT_ID=<your-client-id>
GOOGLE_CLIENT_SECRET=<your-client-secret>
```

### 3. Configure Google Cloud Console
**OAuth Consent Screen:**
- Authorized domains: `skillswap-platform-main.onrender.com`

**Credentials:**
- Authorized redirect URIs: `https://skillswap-platform-main.onrender.com/google/callback`
- Authorized JavaScript origins: `https://skillswap-platform-main.onrender.com`

### 4. Deploy to Render
```bash
git add .
git commit -m "Fix Google OAuth session persistence"
git push origin main
```

---

## 🧪 Testing Checklist

### Local Testing
- [ ] Run `python app.py`
- [ ] Open `http://127.0.0.1:5000`
- [ ] Click "Continue with Google"
- [ ] Complete OAuth flow
- [ ] Verify user is logged in after redirect
- [ ] Refresh page - user should stay logged in
- [ ] Test logout functionality

### Production Testing (Render)
- [ ] Navigate to production URL
- [ ] Click Google login button
- [ ] Complete OAuth flow
- [ ] **CRITICAL**: Verify user stays logged in after page refresh
- [ ] Check browser DevTools > Application > Cookies
- [ ] Verify `session` cookie exists with Secure flag
- [ ] Test all authenticated features work
- [ ] Test logout works

---

## 🔍 Troubleshooting

### Issue: User not logged in after Google OAuth
**Check:**
1. Browser console for errors
2. Network tab - is `/api/auth/user` being called?
3. Application tab - is session cookie set?
4. Render logs - look for "Created new Google OAuth user" message

### Issue: CORS errors
**Solution:**
- Ensure `CORS(app, supports_credentials=True)` is set
- Check Google Cloud Console authorized origins

### Issue: Cookie not persisting
**Solution:**
- Verify `SESSION_COOKIE_SECURE=True` in production
- Ensure HTTPS is being used on Render
- Check SameSite setting is "Lax"

---

## 📝 Files Modified

| File | Lines Changed | Description |
|------|---------------|-------------|
| `app.py` | 14-26 | Session configuration |
| `app.py` | 832-927 | Google OAuth callback + new endpoints |
| `index.html` | 648 | Dynamic BASE_URL |
| `index.html` | 720-765 | Auth check with session |
| `index.html` | 768-791 | Login with credentials |
| `index.html` | 793-816 | Signup with credentials |
| `index.html` | 818-834 | Logout with backend call |

---

## 🎯 Key Improvements

1. ✅ **Server-side session management** - More secure than client-only tokens
2. ✅ **HTTPS-ready cookies** - Proper security flags for production
3. ✅ **Dynamic BASE_URL** - Works on localhost and Render automatically
4. ✅ **Proper logout flow** - Clears both server and client state
5. ✅ **Backward compatible** - Existing users still work via localStorage fallback

---

## 📌 Important Notes

- Session expires after 24 hours (configurable)
- Google OAuth users get auto-created with random password
- Profile photo is pulled from Google account
- Default theme is 'purple' for Google OAuth users

---

**Status**: ✅ Ready for deployment
**Documentation**: See `GOOGLE_OAUTH_FIX_SUMMARY.md` for complete details
