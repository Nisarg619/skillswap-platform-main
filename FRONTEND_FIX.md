# Frontend Authentication Fix - Copy This Section

Replace lines 698-913 in `html_templates/index.html` with this corrected AuthProvider:

```javascript
const AuthProvider = ({ children }) => {
    const [currentUser, setCurrentUser] = useState(null);
    const [userId, setUserId] = useState(localStorage.getItem('skillSwapUserId') || null);
    const [userProfile, setUserProfile] = useState(null);
    const [isAuthReady, setIsAuthReady] = useState(false);

    // Function to fetch user profile
    const fetchUserProfile = useCallback(async (id) => {
        if (!id) return null;
        try {
            const response = await fetch(`${BASE_URL}/api/profile/${id}`, {
                credentials: 'include'
            });
            if (response.ok) {
                const data = await response.json();
                return data;
            } else {
                const errorData = await response.json();
                console.error("Failed to fetch user profile:", errorData.error || response.statusText);
                return null;
            }
        } catch (error) {
            console.error("Network error fetching user profile:", error.message);
            return null;
        }
    }, []);

    // Initial auth check - calls /api/me to check session
    useEffect(() => {
        const checkAuth = async () => {
            try {
                // Check server session first
                const response = await fetch(`${BASE_URL}/api/me`, {
                    credentials: 'include'
                });
                
                if (response.ok) {
                    const data = await response.json();
                    
                    if (data.loggedIn && data.userId) {
                        // User is logged in via session
                        localStorage.setItem('skillSwapUserId', data.userId);
                        setCurrentUser({ uid: data.userId });
                        setUserId(data.userId);
                        setUserProfile(data.userProfile);
                        setIsAuthReady(true);
                        return;
                    }
                }
                
                // Fallback to localStorage
                const storedUserId = localStorage.getItem('skillSwapUserId');
                if (storedUserId) {
                    const profile = await fetchUserProfile(storedUserId);
                    if (profile) {
                        setCurrentUser({ uid: storedUserId });
                        setUserId(storedUserId);
                        setUserProfile(profile);
                    } else {
                        localStorage.removeItem('skillSwapUserId');
                        setUserId(null);
                        setCurrentUser(null);
                        setUserProfile(null);
                    }
                }
                setIsAuthReady(true);
            } catch (error) {
                console.error("Error checking authentication:", error);
                setIsAuthReady(true);
            }
        };
        checkAuth();
    }, [fetchUserProfile]);

    // Login function
    const login = useCallback(async (name, password) => {
        try {
            const response = await fetch(`${BASE_URL}/api/auth/login`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                credentials: 'include',
                body: JSON.stringify({ name, password })
            });
            const data = await response.json();
            if (response.ok) {
                localStorage.setItem('skillSwapUserId', data.userId);
                setCurrentUser({ uid: data.userId });
                setUserId(data.userId);
                setUserProfile(data.userProfile);
                return { success: true, message: data.message };
            } else {
                return { success: false, error: data.error || response.statusText };
            }
        } catch (error) {
            console.error("Network or API error during login:", error.message);
            return { success: false, error: `Network error: ${error.message}. Please ensure the backend server is running.` };
        }
    }, []);

    // Signup function
    const signup = useCallback(async (name, password, location, skillsOffered, skillsWanted, availability, isPublic) => {
        try {
            const response = await fetch(`${BASE_URL}/api/auth/signup`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                credentials: 'include',
                body: JSON.stringify({ name, password, location, skillsOffered, skillsWanted, availability, isPublic })
            });
            const data = await response.json();
            if (response.ok) {
                localStorage.setItem('skillSwapUserId', data.userId);
                setCurrentUser({ uid: data.userId });
                setUserId(data.userId);
                setUserProfile(data.userProfile);
                return { success: true, message: data.message };
            } else {
                return { success: false, error: data.error || response.statusText };
            }
        } catch (error) {
            console.error("Network or API error during signup:", error.message);
            return { success: false, error: `Network error: ${error.message}. Please ensure the backend server is running.` };
        }
    }, []);

    // Logout function
    const logout = useCallback(async () => {
        try {
            await fetch(`${BASE_URL}/api/auth/logout`, {
                method: 'POST',
                credentials: 'include'
            });
        } catch (error) {
            console.error("Error during logout:", error);
        }
        localStorage.removeItem('skillSwapUserId');
        setCurrentUser(null);
        setUserId(null);
        setUserProfile(null);
    }, []);

    // Re-fetch user profile if userId changes
    useEffect(() => {
        if (userId && isAuthReady) {
            fetchUserProfile(userId).then(profile => {
                if (profile) {
                    setUserProfile(profile);
                }
            });
        }
    }, [userId, isAuthReady, fetchUserProfile]);

    const value = {
        currentUser,
        userId,
        userProfile,
        isAuthReady,
        login,
        signup,
        logout,
        fetchUserProfile
    };

    return (
        <AuthContext.Provider value={value}>
            {children}
        </AuthContext.Provider>
    );
};
```

## Key Changes Made:

1. **Removed all JWT token management** - No more accessToken/refreshToken
2. **Uses `/api/me` instead of `/api/auth/user`** - Correct endpoint for session-based auth
3. **All requests use `credentials: 'include'`** - Required for cookies to work
4. **Simplified authentication flow** - No token refresh logic needed
5. **Logout calls backend** - Clears server session

## Google Login Button

Update your Google login button (around line 1174) to:

```javascript
<button
    type="button"
    onClick={() => window.location.href = "https://skillswap-platform-main.onrender.com/login/google"}
    className="w-full py-3 rounded-full font-semibold text-white bg-gradient-to-r from-purple-500 via-purple-600 to-indigo-600 hover:from-purple-600 hover:via-purple-700 hover:to-indigo-700 shadow-lg hover:shadow-purple-500/40 transform hover:scale-[1.02] transition-all duration-300 flex items-center justify-center gap-2"
>
    <span className="text-lg">🔵</span>
    Continue with Google
</button>
```

This will properly redirect to the backend Google OAuth endpoint.
