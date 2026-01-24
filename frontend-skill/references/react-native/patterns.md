<overview>
React Native patterns for building beautiful, performant mobile apps. StyleSheet optimization, native component patterns, and cross-platform design considerations.
</overview>

<styling_fundamentals>
**StyleSheet.create (always use):**
```typescript
import { StyleSheet, View, Text } from 'react-native';

function Card({ title, children }) {
  return (
    <View style={styles.card}>
      <Text style={styles.title}>{title}</Text>
      <View style={styles.content}>{children}</View>
    </View>
  );
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: '#ffffff',
    borderRadius: 12,
    padding: 16,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 8,
    elevation: 3, // Android shadow
  },
  title: {
    fontSize: 18,
    fontWeight: '600',
    color: '#1a1a1a',
    marginBottom: 8,
  },
  content: {
    // Children container
  },
});

export default Card;
```

**Why StyleSheet.create:**
- Style objects are validated at build time
- Styles are only sent to native once
- Enables optimizations (style deduplication)
- Better debugging (style IDs in inspector)
</styling_fundamentals>

<flexbox_layout>
**React Native uses Flexbox (defaults to column):**
```typescript
const styles = StyleSheet.create({
  // Vertical stack (default)
  column: {
    flexDirection: 'column',
    alignItems: 'stretch', // default
  },

  // Horizontal row
  row: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between',
  },

  // Centered content
  centered: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },

  // Flex children
  flexGrow: {
    flex: 1, // Takes remaining space
  },
  flexShrink: {
    flexShrink: 1, // Shrinks when space is tight
  },
  fixedWidth: {
    width: 100, // Fixed, doesn't flex
  },
});
```

**Common layout patterns:**
```typescript
// Header with title and actions
<View style={styles.header}>
  <Text style={styles.headerTitle}>Title</Text>
  <View style={styles.headerActions}>
    <TouchableOpacity>...</TouchableOpacity>
  </View>
</View>

const styles = StyleSheet.create({
  header: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between',
    paddingHorizontal: 16,
    paddingVertical: 12,
  },
  headerTitle: {
    fontSize: 20,
    fontWeight: '600',
  },
  headerActions: {
    flexDirection: 'row',
    gap: 8,
  },
});
```
</flexbox_layout>

<responsive_design>
**Dimensions and platform-specific styles:**
```typescript
import { StyleSheet, Dimensions, Platform } from 'react-native';

const { width, height } = Dimensions.get('window');
const isSmallDevice = width < 375;

const styles = StyleSheet.create({
  container: {
    padding: isSmallDevice ? 12 : 16,
    maxWidth: 600,
    width: '100%',
  },

  // Platform-specific
  shadow: Platform.select({
    ios: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 2 },
      shadowOpacity: 0.1,
      shadowRadius: 8,
    },
    android: {
      elevation: 3,
    },
  }),

  // Inline platform check
  text: {
    fontFamily: Platform.OS === 'ios' ? 'System' : 'Roboto',
    fontSize: 16,
  },
});
```

**useWindowDimensions hook (responsive):**
```typescript
import { useWindowDimensions } from 'react-native';

function ResponsiveGrid({ children }) {
  const { width } = useWindowDimensions();
  const numColumns = width > 768 ? 3 : width > 480 ? 2 : 1;

  return (
    <FlatList
      data={children}
      numColumns={numColumns}
      key={numColumns} // Force re-render on column change
      renderItem={({ item }) => (
        <View style={{ flex: 1 / numColumns, padding: 8 }}>
          {item}
        </View>
      )}
    />
  );
}
```
</responsive_design>

<theming>
**Theme context pattern:**
```typescript
// theme/ThemeContext.tsx
import { createContext, useContext, useState, ReactNode } from 'react';
import { StyleSheet } from 'react-native';

const lightTheme = {
  colors: {
    background: '#ffffff',
    surface: '#f5f5f5',
    text: '#1a1a1a',
    textSecondary: '#666666',
    primary: '#007AFF',
    error: '#FF3B30',
    border: '#e0e0e0',
  },
  spacing: {
    xs: 4,
    sm: 8,
    md: 16,
    lg: 24,
    xl: 32,
  },
  borderRadius: {
    sm: 4,
    md: 8,
    lg: 12,
    full: 9999,
  },
};

const darkTheme = {
  ...lightTheme,
  colors: {
    background: '#1a1a1a',
    surface: '#2a2a2a',
    text: '#ffffff',
    textSecondary: '#999999',
    primary: '#0A84FF',
    error: '#FF453A',
    border: '#3a3a3a',
  },
};

type Theme = typeof lightTheme;

const ThemeContext = createContext<{
  theme: Theme;
  isDark: boolean;
  toggleTheme: () => void;
}>({
  theme: lightTheme,
  isDark: false,
  toggleTheme: () => {},
});

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [isDark, setIsDark] = useState(false);
  const theme = isDark ? darkTheme : lightTheme;

  return (
    <ThemeContext.Provider
      value={{
        theme,
        isDark,
        toggleTheme: () => setIsDark(!isDark),
      }}
    >
      {children}
    </ThemeContext.Provider>
  );
}

export const useTheme = () => useContext(ThemeContext);
```

**Using theme in components:**
```typescript
import { useTheme } from '../theme/ThemeContext';

function ThemedCard({ title, children }) {
  const { theme } = useTheme();

  return (
    <View style={[styles.card, { backgroundColor: theme.colors.surface }]}>
      <Text style={[styles.title, { color: theme.colors.text }]}>
        {title}
      </Text>
      {children}
    </View>
  );
}

const styles = StyleSheet.create({
  card: {
    borderRadius: 12,
    padding: 16,
  },
  title: {
    fontSize: 18,
    fontWeight: '600',
  },
});
```
</theming>

<component_patterns>
**Button component:**
```typescript
import { TouchableOpacity, Text, StyleSheet, ActivityIndicator } from 'react-native';

interface ButtonProps {
  onPress: () => void;
  title: string;
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
}

function Button({
  onPress,
  title,
  variant = 'primary',
  size = 'md',
  loading = false,
  disabled = false,
}: ButtonProps) {
  const isDisabled = disabled || loading;

  return (
    <TouchableOpacity
      onPress={onPress}
      disabled={isDisabled}
      style={[
        styles.base,
        styles[variant],
        styles[size],
        isDisabled && styles.disabled,
      ]}
      activeOpacity={0.7}
    >
      {loading ? (
        <ActivityIndicator color={variant === 'primary' ? '#fff' : '#007AFF'} />
      ) : (
        <Text style={[styles.text, styles[`${variant}Text`], styles[`${size}Text`]]}>
          {title}
        </Text>
      )}
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  base: {
    borderRadius: 8,
    alignItems: 'center',
    justifyContent: 'center',
  },
  primary: {
    backgroundColor: '#007AFF',
  },
  secondary: {
    backgroundColor: '#f0f0f0',
    borderWidth: 1,
    borderColor: '#e0e0e0',
  },
  ghost: {
    backgroundColor: 'transparent',
  },
  sm: {
    paddingVertical: 8,
    paddingHorizontal: 12,
  },
  md: {
    paddingVertical: 12,
    paddingHorizontal: 20,
  },
  lg: {
    paddingVertical: 16,
    paddingHorizontal: 28,
  },
  disabled: {
    opacity: 0.5,
  },
  text: {
    fontWeight: '600',
  },
  primaryText: {
    color: '#ffffff',
  },
  secondaryText: {
    color: '#1a1a1a',
  },
  ghostText: {
    color: '#007AFF',
  },
  smText: {
    fontSize: 14,
  },
  mdText: {
    fontSize: 16,
  },
  lgText: {
    fontSize: 18,
  },
});

export default Button;
```

**Input component:**
```typescript
import { View, TextInput, Text, StyleSheet, TextInputProps } from 'react-native';

interface InputProps extends TextInputProps {
  label?: string;
  error?: string;
}

function Input({ label, error, style, ...props }: InputProps) {
  return (
    <View style={styles.container}>
      {label && <Text style={styles.label}>{label}</Text>}
      <TextInput
        style={[
          styles.input,
          error && styles.inputError,
          style,
        ]}
        placeholderTextColor="#999"
        {...props}
      />
      {error && <Text style={styles.error}>{error}</Text>}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    marginBottom: 16,
  },
  label: {
    fontSize: 14,
    fontWeight: '500',
    color: '#1a1a1a',
    marginBottom: 6,
  },
  input: {
    borderWidth: 1,
    borderColor: '#e0e0e0',
    borderRadius: 8,
    paddingVertical: 12,
    paddingHorizontal: 16,
    fontSize: 16,
    color: '#1a1a1a',
    backgroundColor: '#ffffff',
  },
  inputError: {
    borderColor: '#FF3B30',
  },
  error: {
    fontSize: 12,
    color: '#FF3B30',
    marginTop: 4,
  },
});

export default Input;
```
</component_patterns>

<lists_performance>
**FlatList optimization:**
```typescript
import { FlatList, View, Text, StyleSheet } from 'react-native';

function UserList({ users }) {
  const renderItem = useCallback(
    ({ item }) => <UserCard user={item} />,
    []
  );

  const keyExtractor = useCallback(
    (item) => item.id.toString(),
    []
  );

  const getItemLayout = useCallback(
    (_, index) => ({
      length: ITEM_HEIGHT,
      offset: ITEM_HEIGHT * index,
      index,
    }),
    []
  );

  return (
    <FlatList
      data={users}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      getItemLayout={getItemLayout} // Fixed height items
      initialNumToRender={10}
      maxToRenderPerBatch={10}
      windowSize={5}
      removeClippedSubviews={true}
      ItemSeparatorComponent={() => <View style={styles.separator} />}
      ListEmptyComponent={() => (
        <View style={styles.empty}>
          <Text>No users found</Text>
        </View>
      )}
    />
  );
}

const ITEM_HEIGHT = 72;

const styles = StyleSheet.create({
  separator: {
    height: 1,
    backgroundColor: '#e0e0e0',
  },
  empty: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: 40,
  },
});
```

**SectionList for grouped data:**
```typescript
<SectionList
  sections={[
    { title: 'Active', data: activeUsers },
    { title: 'Inactive', data: inactiveUsers },
  ]}
  renderItem={({ item }) => <UserCard user={item} />}
  renderSectionHeader={({ section }) => (
    <View style={styles.sectionHeader}>
      <Text style={styles.sectionTitle}>{section.title}</Text>
    </View>
  )}
  stickySectionHeadersEnabled={true}
/>
```
</lists_performance>

<animations>
**Animated API basics:**
```typescript
import { Animated, StyleSheet, TouchableOpacity } from 'react-native';
import { useRef, useEffect } from 'react';

function FadeInView({ children }) {
  const fadeAnim = useRef(new Animated.Value(0)).current;

  useEffect(() => {
    Animated.timing(fadeAnim, {
      toValue: 1,
      duration: 300,
      useNativeDriver: true,
    }).start();
  }, []);

  return (
    <Animated.View style={{ opacity: fadeAnim }}>
      {children}
    </Animated.View>
  );
}

function ScaleButton({ onPress, children }) {
  const scaleAnim = useRef(new Animated.Value(1)).current;

  const handlePressIn = () => {
    Animated.spring(scaleAnim, {
      toValue: 0.95,
      useNativeDriver: true,
    }).start();
  };

  const handlePressOut = () => {
    Animated.spring(scaleAnim, {
      toValue: 1,
      friction: 3,
      useNativeDriver: true,
    }).start();
  };

  return (
    <TouchableOpacity
      onPress={onPress}
      onPressIn={handlePressIn}
      onPressOut={handlePressOut}
      activeOpacity={1}
    >
      <Animated.View style={{ transform: [{ scale: scaleAnim }] }}>
        {children}
      </Animated.View>
    </TouchableOpacity>
  );
}
```

**Reanimated for complex animations (recommended):**
```typescript
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
} from 'react-native-reanimated';

function AnimatedCard() {
  const scale = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  const handlePress = () => {
    scale.value = withSpring(scale.value === 1 ? 1.1 : 1);
  };

  return (
    <Pressable onPress={handlePress}>
      <Animated.View style={[styles.card, animatedStyle]}>
        <Text>Tap to animate</Text>
      </Animated.View>
    </Pressable>
  );
}
```
</animations>

<images>
**Image optimization:**
```typescript
import { Image, StyleSheet } from 'react-native';
import FastImage from 'react-native-fast-image';

// Built-in Image (fine for bundled assets)
<Image
  source={require('./assets/logo.png')}
  style={styles.logo}
  resizeMode="contain"
/>

// FastImage for network images (caching, performance)
<FastImage
  source={{
    uri: 'https://example.com/image.jpg',
    priority: FastImage.priority.normal,
    cache: FastImage.cacheControl.immutable,
  }}
  style={styles.avatar}
  resizeMode={FastImage.resizeMode.cover}
/>

const styles = StyleSheet.create({
  logo: {
    width: 120,
    height: 40,
  },
  avatar: {
    width: 48,
    height: 48,
    borderRadius: 24,
  },
});
```

**Responsive images:**
```typescript
import { PixelRatio } from 'react-native';

const scale = PixelRatio.get();
const imageUrl = scale > 2
  ? 'https://example.com/image@3x.jpg'
  : 'https://example.com/image@2x.jpg';
```
</images>

<safe_areas>
**Safe area handling:**
```typescript
import { SafeAreaView, View, StyleSheet } from 'react-native';
import { useSafeAreaInsets } from 'react-native-safe-area-context';

// Simple approach
function Screen({ children }) {
  return (
    <SafeAreaView style={styles.container}>
      {children}
    </SafeAreaView>
  );
}

// Granular control with hook
function CustomScreen({ children }) {
  const insets = useSafeAreaInsets();

  return (
    <View style={[
      styles.container,
      {
        paddingTop: insets.top,
        paddingBottom: insets.bottom,
        paddingLeft: insets.left,
        paddingRight: insets.right,
      }
    ]}>
      {children}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#ffffff',
  },
});
```
</safe_areas>

<anti_patterns>
**Avoid these React Native patterns:**

```typescript
// BAD: Inline styles (no optimization)
<View style={{ padding: 16, backgroundColor: '#fff' }}>

// GOOD: StyleSheet.create
<View style={styles.container}>

// BAD: Anonymous functions in render
<FlatList
  renderItem={({ item }) => <Card item={item} />} // New function every render
/>

// GOOD: useCallback or extracted function
const renderItem = useCallback(({ item }) => <Card item={item} />, []);
<FlatList renderItem={renderItem} />

// BAD: Nested Views for spacing
<View>
  <View style={{ marginBottom: 8 }} />
  <Text>Content</Text>
</View>

// GOOD: Gap or margin on elements
<View style={{ gap: 8 }}>
  <Text>Content</Text>
</View>

// BAD: Large inline arrays in style prop
<View style={[styles.a, styles.b, styles.c, { marginTop: 10 }]}>

// GOOD: Combine styles or use conditional styling
const combinedStyle = useMemo(
  () => [styles.a, styles.b, styles.c, { marginTop: 10 }],
  []
);
<View style={combinedStyle}>

// BAD: Not using useNativeDriver for animations
Animated.timing(opacity, {
  toValue: 1,
  duration: 300,
  // Missing useNativeDriver!
}).start();

// GOOD: Enable native driver when possible
Animated.timing(opacity, {
  toValue: 1,
  duration: 300,
  useNativeDriver: true,
}).start();
```
</anti_patterns>

<performance_tips>
- Use `useMemo` and `useCallback` for expensive computations and callbacks
- Enable Hermes engine for faster startup and lower memory usage
- Use `removeClippedSubviews` on long lists
- Avoid passing new object/array literals as props
- Use `react-native-fast-image` for network images
- Profile with Flipper and React DevTools
- Use `InteractionManager.runAfterInteractions` for non-critical work
- Minimize bridge calls (batch operations)
</performance_tips>
