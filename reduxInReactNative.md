# In this challenge clients decides to open most of the app features for all of the customers except some specific parts. So he wants to remove login feature from the application and request login only if customer wants to use only one of the navigation.

__first let's design our store.js file which includes userReducer__

```js
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './userSlice';

const store = configureStore({
  reducer: {
    user: userReducer,
  },
});

export default store;
```

__Now it's the time for most import file which is userSlice__

```js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// --- IMPORTANT: Configure your API base URL here ---
const API_BASE_URL = 'http://10.0.2.2:3000'; // For Android Emulator
// For iOS simulator/device or physical Android, use your actual local IP address (e.g., 'http://192.168.1.X:3000')

export const loginUser = createAsyncThunk(
  'user/login',
  async ({ username, password }: { username: string, password: string }, { rejectWithValue }) => {
    try {
      const response = await fetch(`${API_BASE_URL}/users`);
      if (!response.ok) {
        throw new Error(`Backend error: ${response.status}`);
      }
      const users = await response.json();

      const foundUser = users.find(
        (user: any) => user.username === username && user.password === password
      );

      if (!foundUser) {
        return rejectWithValue('Invalid username or password.');
      }
      return foundUser;

    } catch (error: any) {
      console.error("Login API error:", error);
      return rejectWithValue(error.message || 'An unexpected error occurred during login.');
    }
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState: {
    isAuthenticated: false,
    userName: null,
    userId: null,
    isLoading: false, // This loading state is for login/API calls, not initial app load
    error: null,
  },
  reducers: {
    logout(state) {
      state.isAuthenticated = false;
      state.userName = null;
      state.userId = null;
      state.error = null;
      state.isLoading = false;
    },
    setLoading(state, action) { // Used for overall app initial loading state
      state.isLoading = action.payload;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(loginUser.pending, (state) => {
        state.isLoading = true;
        state.error = null;
      })
      .addCase(loginUser.fulfilled, (state, action) => {
        state.isLoading = false;
        state.isAuthenticated = true;
        state.userName = action.payload.fullName;
        state.userId = action.payload.id;
        state.error = null;
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.isLoading = false;
        state.isAuthenticated = false;
        state.userName = null;
        state.userId = null;
        state.error = action.payload as string || 'Failed to login. Please try again.';
      });
  },
});

export const { logout, setLoading } = userSlice.actions;
export default userSlice.reducer;
```

__And this is the LoginScreen__

```js
import React, { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, Alert, ActivityIndicator } from 'react-native';
import { useDispatch, useSelector } from 'react-redux';
import { loginUser } from '../redux/userSlice';

const LoginScreen = () => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const dispatch = useDispatch();
  const isLoading = useSelector((state: any) => state.user.isLoading);
  const error = useSelector((state: any) => state.user.error);

  const handleLogin = async () => {
    if (username.trim() === '' || password.trim() === '') {
        Alert.alert('Input Error', 'Please enter both username and password.');
        return;
    }

    try {
        await dispatch(loginUser({ username, password })).unwrap();
    } catch (err: any) {
        Alert.alert('Login Failed', err);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Login to FitnessApp</Text>
      <TextInput
        style={styles.input}
        placeholder="Username"
        value={username}
        onChangeText={setUsername}
        autoCapitalize="none"
      />
      <TextInput
        style={styles.input}
        placeholder="Password"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
      />
      {isLoading ? (
        <ActivityIndicator size="large" color="#0000ff" />
      ) : (
        <Button title="Login" onPress={handleLogin} />
      )}
      {error && <Text style={styles.errorText}>{error}</Text>}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 30,
    color: '#333',
  },
  input: {
    width: '100%',
    padding: 15,
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    marginBottom: 15,
    backgroundColor: '#fff',
    fontSize: 16,
  },
  errorText: {
    color: 'red',
    marginTop: 10,
    textAlign: 'center',
  },
});

export default LoginScreen;
```

__So we add this part of the codes to our navigation components__

```js
 const isAuthenticated = useSelector((state: any) => state.user.isAuthenticated);
  const isLoading = useSelector((state: any) => state.user.isLoading);
  const dispatch = useDispatch();

  // Simulate initial authentication check (e.g., checking a stored token,
  // or fetching user session from backend on app start)
  useEffect(() => {
    // In a real app, you'd likely dispatch an async thunk here to verify a token
    // or fetch initial user data. For this example, we'll just simulate a delay.
    setTimeout(() => {
      dispatch(setLoading(false)); // Once check is done, set loading to false
    }, 1500); // Simulate 1.5 seconds of loading
  }, [dispatch]);
  ```

  __And we just need to put this codes in front of our navigaion components__

```js
{isAuthenticated ? (
        // User is logged in, show authenticated screens
        <>
        <Stack.Screen name="Profile" component={UserProfileScreen} />
        <Stack.Screen name="Settings" component={SettingsScreen} />
        </>
    )
```



