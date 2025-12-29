# AI Coding Agent Instructions - KeylessRN

## Quick Start for New Agents

**KeylessRN** is a multi-flavor React Native app (iOS/Android) for smart home lock access and device management. Key facts:
- **Redux + Redux-Saga** state management with persistent storage
- **Multi-flavor support** (8 white-label brands: keyless, progress, kairos, nhr, firstkey, camillo, coveyhomes, bellanet)
- **Role-based access**: Resident, Manager, InstallerV3, InstallerV5, Agent
- **MQTT communication** for real-time device events
- **Offline-first**: MMKV storage, async state sync

## Architecture Patterns You'll Encounter

### Redux + Saga Data Flow
```
User Action → Saga (side effects) → Reducer (state) → Component re-render
```

Files follow naming pattern: `/src/{Feature}/actions.ts`, `/redux/reducer.ts`, `/sagas.ts`

**Key files to understand Redux setup:**
- [`src/common/store/index.ts`](src/common/store/index.ts) - Store initialization, saga middleware, persistence config (whitelist: app, home, login, userGuide, faceIdReducer, shortRangeOfflineActivity, dashboard)
- [`src/common/store/combineReducers.ts`](src/common/store/combineReducers.ts) - All reducers combined
- [`src/common/store/typeSafe.ts`](src/common/store/typeSafe.ts) - Custom action/reducer utilities
- [`src/common/loaderRedux/`](src/common/loaderRedux/) - Global loader state pattern (startLoadingAction, successLoadingAction, failedLoadingAction with keyed loaders)

## Overall Architecture Pattern

KeylessRN follows a **sophisticated, enterprise-grade Flux/Redux architecture with plugin-based modular design** for scalability, maintainability, and performance in IoT smart home management.

### Architecture Layers

```
PRESENTATION LAYER
├── React Native Components (Views)
│   ├── Home Module (Dashboard, Device Controls)
│   ├── Authentication Module (Login, Biometric)
│   ├── Camera Module (Video Streaming)
│   ├── Installer Module (Device Setup, Testing)
│   └── Settings/Manage Module (Configuration)
        ↓
STATE MANAGEMENT LAYER
├── Redux Store (Single Source of Truth)
│   ├── Actions (User Intentions/Commands)
│   ├── Reducers (Pure State Transformers)
│   ├── Sagas (Side Effect Management)
│   └── Selectors (State Access Layer)
        ↓
BUSINESS LOGIC LAYER
├── Plugin Architecture
│   ├── PersistencePlugin (Local Storage Interface)
│   ├── BluetoothPlugin (BLE Device Communication)
│   ├── MQTTPlugin (Real-time Device Updates)
│   └── CalibrationPlugin (Device Configuration)
        ↓
DATA ACCESS LAYER
├── External Integrations
│   ├── REST APIs (Backend Services)
│   ├── Firebase (Analytics, Crash Reporting, Remote Config)
│   ├── AWS IoT (Device Shadows, MQTT)
│   ├── Bluetooth LE (Direct Device Communication)
│   └── Local Storage (AsyncStorage, Redux Persist)
```

### Core Architecture Components

**1. State Management (Redux/Flux Pattern)**
```typescript
// Unidirectional Data Flow
Action → Reducer → Store → Component → Action

// Example Flow
dispatch(lockDeviceAction({deviceId, lockMac}))
  → deviceReducer updates state
  → Component re-renders with new state
  → User sees updated UI
```

**2. Plugin Architecture**
```typescript
// Interface-based plugin system
abstract class PluginInterface {
  abstract method(): Promise<any>;
}
class ConcretePlugin extends PluginInterface {
  async method(): Promise<any> {
    // Implementation
  }
}
```

**3. Modular Feature Organization**
```
src/
├── Home/                    # Main dashboard & device controls
├── Authentication/          # Login, biometric auth
├── CameraHome/              # Video streaming features
├── HubInstaller/            # Device installation workflows
├── Intercom/                # Communication features
├── MoveInMoveOut/           # Tenant management
└── common/                  # Shared utilities & components
```

### Design Patterns Implementation

**1. Observer Pattern**
- Redux store subscriptions
- React component lifecycle hooks
- MQTT event listeners
- Bluetooth device state changes

**2. Strategy Pattern**
```typescript
// Thermostat calibration strategies
class Conventional extends CalibrationInterface { /* ... */ }
class HeatPump extends CalibrationInterface { /* ... */ }
class Boiler extends CalibrationInterface { /* ... */ }
```

**3. Factory Pattern**
- SSL certificate pinning factory
- Device type factories
- Navigation factory patterns

**4. Adapter Pattern**
- BLE library adapters
- Platform-specific implementations
- Third-party service integrations

**5. Command Pattern**
- Redux actions as commands
- Device operation commands
- Undo/redo functionality

**6. Singleton Pattern**
- Global Redux store
- MQTT connection manager
- Logger instances

### Data Flow Architecture

**Request Flow Example: Lock Device**

```
1. User taps "Lock" button
   └── Component dispatches lockDeviceAction()
2. Redux Action
   └── { type: 'LOCK_DEVICE_REQUEST', payload: {deviceId, lockMac} }
3. Redux-Saga intercepts action
   └── Performs side effects (BLE communication, API calls)
4. Saga dispatches success/failure actions
   └── Updates Redux store state
5. Component re-renders
   └── UI reflects new lock state
6. Optional: MQTT receives confirmation
   └── Real-time state synchronization
```

**Data Persistence Strategy**
```
Local Cache (Redux Persist)
    ↓
AsyncStorage (React Native)
    ↓
Remote API (Backend Sync)
    ↓
Cloud Storage (AWS/Firebase)
```

### Technology Stack

**Core Framework**
- React Native: 0.74.3
- React: ^18.3.1
- TypeScript: 5.4.5
- Flow: ^0.162.0

**State Management**
- Redux: ^5.0.1
- React-Redux: ^9.1.2
- Redux-Saga: ^1.3.0
- Redux-Persist: ^6.0.0
- Immer: ^10.1.1

**Navigation & UI**
- React Navigation v6
- React Native Paper: ^5.12.3
- React Native Vector Icons: ^10.1.0
- React Native SVG: ^15.2.0

**Device Integration**
- React Native BLE PLX: ^3.1.2
- AWS IoT SDK: ^2.2.13
- React Native Permissions: 4.1.5

**Backend & Services**
- Firebase Suite (Analytics, Crashlytics, Messaging, Remote Config)
- Axios: ^1.6.8
- AsyncStorage
- MQTT (Real-time device updates)

### Architectural Principles

**1. Separation of Concerns**
- **Presentation**: React components handle only UI rendering
- **Business Logic**: Sagas handle complex operations and side effects
- **Data**: Reducers manage state transformations
- **Integration**: Plugins abstract external dependencies

**2. Single Responsibility**
- Each module handles one domain area (Home, Auth, Installer, etc.)
- Components have single, clear purposes
- Reducers handle specific state slices
- Sagas manage distinct workflows

**3. Dependency Inversion**
- High-level modules don't depend on low-level modules
- Plugin interfaces abstract concrete implementations
- Dependency injection through Redux store and configuration

**4. Open/Closed Principle**
- Extensible through plugin system
- New device types via strategy pattern
- Modular feature addition without core changes

### Security Architecture

**Data Protection**
- SSL Pinning (certificate verification)
- Keychain Storage (sensitive credentials)
- Biometric Authentication (face ID, fingerprint)
- Token Management (refresh token rotation)

**Device Security**
- BLE Encryption (device communication)
- Device Authentication (mutual verification)
- Offline Security (cached encrypted data)

### Performance Optimizations

**State Management**
- Selective Re-rendering (memoized components)
- Normalized State (flat structure for fast lookups)
- Lazy Loading (code splitting, dynamic imports)
- Memoized Selectors (reselect library)

**Device Communication**
- Connection Pooling (BLE/MQTT)
- Retry Logic (exponential backoff)
- Background Processing (app state aware)
- Caching (device states, configurations)

### Development & Testing

**Code Quality**
- ESLint (code standards)
- Flow/TypeScript (type safety)
- Prettier (code formatting)
- Jest (unit testing)

**Debugging Tools**
- Redux DevTools (state inspection)
- Reactotron (Redux/network monitoring)
- Firebase Crashlytics (error tracking)
- Console Logging (logger utility)

**Example action structure:**
```typescript
// Type-safe action creator using custom 'action()' utility
export const storePropertyDetailsAction = (payload: PropertyType): any => 
  action('Feature/storePropertyDetailsAction', payload);

// Saga handles side effects
function* fetchPropertySaga() {
  yield put(startLoadingAction('PropertyKey'));
  try {
    const data = yield call(api.getProperty);
    yield put(storePropertyDetailsAction(data));
  } catch (error) {
    yield put(failedLoadingAction('PropertyKey', error.message));
  }
}
```

### Flavor-Based Configuration
8 flavors share code but have distinct branding/behavior. Flavor detection:
- **Build time**: Android Gradle flavors (flavorDefault, flavorProgress, etc.)
- **Runtime**: `Config.FLAVOUR` from [`config.ts`](config.ts) (environment-dependent)
- **Shared assets**: `/assets/tuya/officialSDK/{flavor}/` for native SDK configs

## TypeScript Coding Standards

All code must follow strict TypeScript standards to catch errors at compile-time and ensure type safety. The project uses `strict: true` in `tsconfig.json`.

### 1. Enable Strict Type Checking
**Rule**: TypeScript is configured with `strict: true` enabling all strict type-checking options.

**Why**: Catches errors at compile time, reducing runtime issues and improving code quality.

Key compiler options in [tsconfig.json](tsconfig.json):
```json
{
  "compilerOptions": {
    "strict": true,
    "target": "esnext",
    "module": "esnext",
    "jsx": "react-native",
    "moduleResolution": "node",
    "allowJs": false,
    "noEmit": true,
    "isolatedModules": true,
    "strictNullChecks": true,
    "noImplicitAny": false,
    "resolveJsonModule": true,
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "moduleSuffixes": [".android", ".ios", ""]
  }
}
```

### 2. Explicit Types for Variables, Parameters, and Return Types
**Rule**: Always declare explicit types for variables, function parameters, and return types.

**Incorrect:**
```typescript
const add = (a, b) => a + b;
```

**Correct:**
```typescript
const add = (a: number, b: number): number => a + b;
```

### 3. Use interface for Objects, type for Primitives and Unions
**Rule**: Use `interface` for object types that may be extended. Use `type` for primitives, unions, or intersections.

**Incorrect:**
```typescript
const user: { id: number; name: string } = { id: 1, name: "Alice" };
```

**Correct:**
```typescript
interface User {
  id: number;
  name: string;
}
type Status = "active" | "inactive";
const user: User = { id: 1, name: "Alice" };
const status: Status = "active";
```

### 4. Use React.FC for Functional Components
**Rule**: Use `React.FC<Props>` for React functional components to enforce prop typing.

**Incorrect:**
```typescript
const Button = ({ title, onPress }) => (
  <Pressable onPress={onPress}><Text>{title}</Text></Pressable>
);
```

**Correct:**
```typescript
interface ButtonProps {
  title: string;
  onPress: () => void;
}
const Button: React.FC<ButtonProps> = ({ title, onPress }) => (
  <Pressable onPress={onPress}><Text>{title}</Text></Pressable>
);
```

### 5. Type State with useState
**Rule**: Explicitly type `useState` calls to avoid type inference issues.

**Incorrect:**
```typescript
const [user, setUser] = useState({ id: 1, name: "Alice" });
```

**Correct:**
```typescript
interface User {
  id: number;
  name: string;
}
const [user, setUser] = useState<User>({ id: 1, name: "Alice" });
```

### 6. Type useRef Correctly
**Rule**: Specify the type for `useRef` to reflect the referenced element, typically allowing null.

**Incorrect:**
```typescript
const inputRef = useRef(null);
```

**Correct:**
```typescript
const inputRef = useRef<TextInput | null>(null);
```

### 7. Type FlatList Data and keyExtractor
**Rule**: Define a type for FlatList data and provide a keyExtractor function with unique keys.

**Incorrect:**
```typescript
<FlatList
  data={[{ id: "1", name: "Alice" }, { id: "2", name: "Bob" }]}
  keyExtractor={(item) => item.id}
/>;
```

**Correct:**
```typescript
interface Item {
  id: string;
  name: string;
}
const data: Item[] = [
  { id: "1", name: "Alice" },
  { id: "2", name: "Bob" }
];
<FlatList
  data={data}
  keyExtractor={(item) => item.id}
  renderItem={({ item }) => <Text>{item.name}</Text>}
/>;
```

### 8. Define Types for API Responses
**Rule**: Always define specific types for API responses instead of using `any`.

**Incorrect:**
```typescript
const fetchData = async (): Promise<any> => {
  const response = await fetch("https://api.example.com/data");
  return response.json();
};
```

**Correct:**
```typescript
interface DataResponse {
  id: number;
  value: string;
}
const fetchData = async (): Promise<DataResponse> => {
  const response = await fetch("https://api.example.com/data");
  return response.json();
};
```

### 9. Use Enums for Constant Values
**Rule**: Use `enum` for sets of related constants to avoid magic strings.

**Incorrect:**
```typescript
const setRole = (role: "admin" | "user") => {};
```

**Correct:**
```typescript
enum Role {
  Admin = "admin",
  User = "user",
}
const setRole = (role: Role) => {};
```

### 10. Avoid any, Prefer unknown
**Rule**: Avoid `any` to maintain type safety. Use `unknown` when type is uncertain and narrow it with type checks.

**Incorrect:**
```typescript
const process = (data: any) => data.toUpperCase();
```

**Correct:**
```typescript
const process = (data: unknown) => {
  if (typeof data === "string") {
    return data.toUpperCase();
  }
  return null;
};
```

### 11. Use as const for Immutable Literals
**Rule**: Use `as const` for literal objects or arrays to make them immutable and infer literal types.

**Incorrect:**
```typescript
const config = { apiUrl: "https://api.example.com" };
```

**Correct:**
```typescript
const config = { apiUrl: "https://api.example.com" } as const;
```

### 12. Handle Nullable Types with strictNullChecks
**Rule**: With `strictNullChecks` enabled, explicitly handle null and undefined in types using type guards.

**Example:**
```typescript
type Response = string | null;
const handleResponse = (response: Response) => {
  if (response !== null) {
    return response.toUpperCase();
  }
  return "";
};
```

### 13. Organize Types in Separate Files
**Rule**: Define reusable types and interfaces in dedicated files (e.g., `types/` directory) and import them.

**Example:**
```typescript
// types/user.ts
export interface User {
  id: number;
  name: string;
}

// Component file
import { User } from "../types/user";
const user: User = { id: 1, name: "Alice" };
```

### 14. Use unknown for External Data
**Rule**: Use `unknown` for data from external sources (APIs, user input) and validate before use.

**Example:**
```typescript
async function fetchData() {
  const response: unknown = await fetch("https://api.example.com").then(res => res.json());
  if (typeof response === "object" && response !== null && "id" in response) {
    const data = response as { id: number };
    console.log(data.id);
  }
}
```

### 15. Enforce Consistent File Naming
**Rule**: Use `.ts` for TypeScript files and `.tsx` for files containing JSX.

**Example:**
- `Component.tsx` for React components
- `utils.ts` for non-React utilities
- `types.ts` for type definitions

### 16. Type Third-Party Library Integrations
**Rule**: Use `@types/*` packages for third-party libraries or write custom declaration files if types unavailable.

**Example:**
```typescript
// For a library without types
declare module 'some-library' {
  export function doSomething(value: string): string;
}
```

### 17. Use React.ReactNode for Flexible Children
**Rule**: Use `React.ReactNode` for component children to allow any valid JSX content.

**Example:**
```typescript
interface ContainerProps {
  children: React.ReactNode;
}
const Container: React.FC<ContainerProps> = ({ children }) => (
  <View>{children}</View>
);
```

### 18. Handle Async Functions with Proper Types
**Rule**: Use `Promise<T>` for async function return types and handle errors with try-catch.

**Example:**
```typescript
interface User {
  id: number;
  name: string;
}
const fetchUser = async (id: number): Promise<User> => {
  try {
    const response = await fetch(`https://api.example.com/users/${id}`);
    return response.json();
  } catch (error) {
    throw new Error('Failed to fetch user');
  }
};
```

### 19. Type Navigation Props for React Navigation
**Rule**: Use `@react-navigation/native` types to define navigation and route props for screens.

**Example:**
```typescript
import { NativeStackScreenProps } from '@react-navigation/native-stack';

type RootStackParamList = {
  Home: undefined;
  Profile: { userId: number };
};

type ProfileScreenProps = NativeStackScreenProps<RootStackParamList, 'Profile'>;

const ProfileScreen: React.FC<ProfileScreenProps> = ({ navigation, route }) => {
  const { userId } = route.params;
  return <View><Text>User ID: {userId}</Text></View>;
};
```

## React Native + TypeScript Naming Conventions

Consistent naming across the codebase improves readability and reduces confusion. Follow these conventions strictly.

### 1. PascalCase
**Use for:**
- Component names
- Screens
- Providers
- Custom hooks (start with `use`)
- Types & Interfaces
- Enums

**Correct Example:**
```typescript
const MyComponent = () => {
  return <View />;
};
const PropertyCard = () => {};
const HomeScreen = () => {};
const AuthProvider = () => {};
const useAuth = () => {}; // Custom hook
interface UserProfile {}
enum PropertyStatus {
  Available,
  Rented,
}
```

**Incorrect Example:**
```typescript
const mycomponent = () => {
  return <View />;
};
```

### 2. camelCase
**Use for:**
- Variables
- Functions
- Methods
- Local state
- Parameters
- Utility/helper functions
- Redux actions
- Sagas

**Correct Example:**
```typescript
const myMethod = () => {};
const myValue = "RentlySmartHome";
const isLoading = true;
const userProfile = null;
const handleSubmit = () => {};
const fetchProperties = async () => {};
const formatPhoneNumber = (phone: string) => {};
const storePropertyDetailsAction = (payload: PropertyType) => action(...);
```

**Incorrect Example:**
```typescript
const MyMethod = () => {};
const my_value = "RentlySmartHome";
```

### 3. CONSTANT_CASE (UPPER_SNAKE_CASE)
**Use for:**
- Global constants
- Immutable config values
- Enum values (if all caps)
- Redux action type strings

**Correct Example:**
```typescript
const API_TIMEOUT = 30000;
const BASE_URL = "https://api.example.com";
const MAX_FILE_SIZE = 10 * 1024 * 1024;
const DEFAULT_COUNTRY = "US";
export const SUPPORTED_LOCALES = ["en", "es", "fr"] as const;
const PROPERTY_DETAIL_ACTION = 'Feature/storePropertyDetailsAction';
```

**Incorrect Example:**
```typescript
const apiTimeout = 30000;
const baseUrl = "https://api.example.com";
```

### 4. File & Folder Naming

| Type | Convention | Example |
|------|-----------|---------|
| Component | PascalCase | `PropertyCard.tsx` |
| Screen | PascalCase | `HomeScreen.tsx`, `LoginScreen.tsx` |
| Custom Hook | camelCase | `useAuth.ts`, `useProperties.ts` |
| Utility / Helper | camelCase | `dateUtils.ts`, `formatters.ts` |
| Service / API | camelCase | `apiClient.ts`, `authService.ts` |
| Constants File | camelCase | `constants.ts`, `config.ts` |
| Types/Interfaces | PascalCase or camelCase | `UserTypes.ts`, `types.ts` |
| Styles | `.styles.ts` | `PropertyCard.styles.ts` |
| Redux Actions | camelCase (file) | `actions.ts` |
| Redux Reducer | camelCase (file) | `reducer.ts` |
| Redux Saga | camelCase (file) | `sagas.ts` |
| API Functions | camelCase (file) | `apis.ts` |
| Test Files | `.test.tsx` or `.spec.tsx` | `PropertyCard.test.tsx` |

### 5. Folder Structure & Naming

**Component folders:** PascalCase
```
src/
  common/
    components/
      PropertyCard/          # Folder: PascalCase
        PropertyCard.tsx     # Component: PascalCase
        PropertyCard.styles.ts
        index.ts
      FTextInputWithLabel/
        FTextInputWithLabel.tsx
        FTextInputWithLabel.styles.ts
        index.ts
```

**Feature/module folders:** kebab-case (lowercase with hyphens)
```
src/
  home-screen/             # Feature folder: kebab-case
    home-tab/
      redux/
        actions.ts         # Redux files: camelCase
        reducer.ts
        sagas.ts
      apis.ts              # API file: camelCase
      HomeTab.tsx          # Screen: PascalCase
  property-details/
  lock-management/
  hub-installer/
```

**Redux folder structure:** Feature folder → `redux/` → specific reducer files
```
src/
  lock-management/
    redux/
      actions.ts           # `storePropertyDetailsAction()`, `fetchPropertyDetailsAction()`
      reducer.ts           # Property reducer logic
      sagas.ts             # Property side effects
    apis.ts                # Property API calls
    LockDetail.tsx         # Component
```

**Common utilities:** kebab-case
```
src/
  common/
    store/                 # Core Redux setup
    api-wrapper/           # Centralized axios
    components/            # Reusable components
    user-management/       # Auth & role helpers
    logger/                # Logging utility
    constants.ts
```

### 6. Redux-Specific Naming

**Actions:**
```typescript
// camelCase function name, follows pattern: {verb}{Feature}{Action}
export const storePropertyDetailsAction = (payload: PropertyType): any =>
  action('Feature/storePropertyDetailsAction', payload);

export const fetchPropertyDetailsAction = (payload: { id: string }): any =>
  action('Feature/fetchPropertyDetailsAction', payload);

// Action type string: PascalCase feature + action type
export const PROPERTY_DETAILS_SUCCESS = 'PropertyDetails/Success';
```

**Reducers:**
```typescript
// File: camelCase
// File path: src/Home/homeReducer.ts

// Reducer function: lowercase, typically named after feature or 'reducer'
const homeReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'Property/StoreProperty':
      return { ...state, propertyDetails: action.payload };
    default:
      return state;
  }
};
```

**Sagas:**
```typescript
// File: camelCase (sagas.ts)

// Saga function: camelCase, follows pattern: {feature}{verb}Saga
function* fetchPropertyDetailsSaga(action: any) {
  // Implementation
}

export function* watchFetchPropertyDetailsSaga() {
  yield takeEvery(fetchPropertyDetailsAction, fetchPropertyDetailsSaga);
}
```

**Selectors:**
```typescript
// Selector function: camelCase, typically starts with 'get' or 'select'
export const getPropertyDetails = (state: RootState) => state.home.propertyDetails;
export const selectUserRole = (state: RootState) => state.app.userRole;
export const loaderSelector = (loaderKey: string) => (state: RootState) =>
  state.loader.loaders[loaderKey];
```

### 7. TypeScript Interface/Type Naming

**Request/Response types:**
```typescript
// API request: {Feature}Request
interface PropertyDetailsRequest {
  propertyId: string;
}

// API response: {Feature}Response
interface PropertyDetailsResponse {
  id: string;
  name: string;
  address: string;
}

// Component props: {ComponentName}Props
interface PropertyCardProps {
  property: Property;
  onPress: () => void;
}

// Redux state: {Feature}State
interface HomeState {
  propertyDetails: Property | null;
  loading: boolean;
  error: string | null;
}
```

**Generic types:**
```typescript
// Status enums: {Feature}Status
enum PropertyStatus {
  Available = 'available',
  Rented = 'rented',
  Maintenance = 'maintenance',
}

// Role enums: {RoleType}
enum UserRole {
  Resident = 'resident',
  Manager = 'manager',
  Installer = 'installer',
}
```

### 8. Variable Naming Patterns

**Boolean variables:** Start with `is`, `has`, or `can`
```typescript
const isLoading = true;
const hasPermission = false;
const canEdit = user.role === 'manager';
const isOnline = state.connectivity.online;
const hasError = error !== null;
```

**Event handlers:** Start with `handle` or `on`
```typescript
const handleSubmit = () => {};
const handlePropertySelect = (id: string) => {};
const onPropertyPress = () => {};
const onTextChange = (text: string) => {};
```

**Async functions:** Use `fetch` or `get` prefix
```typescript
const fetchPropertyDetails = async () => {};
const getDeviceStatus = async () => {};
const loadUserProfile = async () => {};
```

**Mapping/Transform functions:** Start with `map`, `format`, or `parse`
```typescript
const mapPropertiesToUI = (properties: Property[]) => [];
const formatPhoneNumber = (phone: string) => string;
const parseErrorResponse = (error: unknown) => ErrorType;
```

## React Native Component & Rendering Rules

### Use Functional Components with Hooks

Always prefer functional components over class components unless there's a specific need.

**Incorrect (Class Component):**
```typescript
class MyComponent extends React.Component {
  state = { count: 0 };
  render() {
    return <Text>{this.state.count}</Text>;
  }
}
```

**Correct (Functional with Hooks):**
```typescript
const MyComponent: React.FC = () => {
  const [count, setCount] = useState(0);
  return <Text>{count}</Text>;
};
```

### Keep Components Small and Focused

Each component should do one thing well. Split large components into smaller reusable ones.

**Incorrect (Too Many Responsibilities):**
```typescript
const Profile: React.FC = () => (
  <View>
    <Image source={{ uri: "..." }} />
    <Text>Name</Text>
    <Text>Bio</Text>
  </View>
);
```

**Correct (Splitting UI Elements):**
```typescript
const Profile: React.FC = () => (
  <View>
    <ProfilePicture />
    <ProfileDetails />
  </View>
);
```

### Use Meaningful Component Names

Component names should describe their purpose, not be generic.

**Incorrect:**
```typescript
const MyComponent = () => <View />;
```

**Correct:**
```typescript
const UserCard: React.FC = () => <View />;
```

### Avoid Inline Styles – Use StyleSheet.create

Inline styles reduce performance. Use `StyleSheet.create()` for optimization.

**Incorrect:**
```typescript
const MyComponent: React.FC = () => (
  <View style={{ backgroundColor: "white", padding: 10 }} />
);
```

**Correct:**
```typescript
const styles = StyleSheet.create({
  container: {
    backgroundColor: "white",
    padding: 10,
  },
});
const MyComponent: React.FC = () => <View style={styles.container} />;
```

### Use Flexbox for Layout

Avoid fixed widths and heights. Use flexbox for responsive layouts.

**Incorrect:**
```typescript
<View style={{ width: 300, height: 200 }}>
  <Text>Hello</Text>
</View>
```

**Correct:**
```typescript
<View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
  <Text>Hello</Text>
</View>
```

### Use Platform-Specific Code Properly

Use the Platform API for platform-specific styles or logic, not inline checks in JSX.

**Incorrect:**
```typescript
<Text style={{ fontSize: Platform.OS === "ios" ? 20 : 18 }}>Hello</Text>
```

**Correct:**
```typescript
const styles = StyleSheet.create({
  text: {
    fontSize: Platform.OS === "ios" ? 20 : 18,
  },
});
<Text style={styles.text}>Hello</Text>
```

### Optimize Lists with FlatList

Use FlatList for large lists instead of ScrollView to improve performance.

**Incorrect (ScrollView for Large Lists):**
```typescript
<ScrollView>
  {data.map((item) => (
    <Text key={item.id}>{item.name}</Text>
  ))}
</ScrollView>
```

**Correct (FlatList with keyExtractor):**
```typescript
interface Item {
  id: string;
  name: string;
}
const data: Item[] = [
  { id: "1", name: "John" },
  { id: "2", name: "Doe" }
];
const MyComponent: React.FC = () => (
  <FlatList
    data={data}
    keyExtractor={(item) => item.id}
    renderItem={({ item }) => <Text>{item.name}</Text>}
  />
);
```

### Avoid Unnecessary Renders

Use `React.memo` for pure components to prevent re-renders when props don't change. Use `useCallback` to memoize functions.

**Incorrect (Creates New Function Every Render):**
```typescript
const MyComponent: React.FC = () => {
  const onPress = () => console.log("Pressed");
  return <ChildComponent onPress={onPress} />;
};
```

**Correct (Memoized Function and Component):**
```typescript
const MyComponent: React.FC = () => {
  const onPress = useCallback(() => console.log("Pressed"), []);
  return <ChildComponent onPress={onPress} />;
};

const ChildComponent: React.FC<{ onPress: () => void }> = React.memo(({ onPress }) => {
  return (
    <Pressable onPress={onPress}>
      <Text>Click</Text>
    </Pressable>
  );
});
```

### Avoid Component Definitions Inside Render

Defining components inside the render function causes unnecessary re-renders and state loss.

**Incorrect:**
```typescript
const Parent: React.FC = () => {
  const [count, setCount] = useState(0);
  const Child: React.FC<{ name: string }> = ({ name }) => <Text>{name}</Text>;
  return (
    <>
      <Child name="John" />
      <Button title="Increment" onPress={() => setCount(count + 1)} />
    </>
  );
};
```

**Correct:**
```typescript
interface ChildProps {
  name: string;
}
const Child: React.FC<ChildProps> = React.memo(({ name }) => {
  return <Text>{name}</Text>;
});

const Parent: React.FC = () => {
  const [count, setCount] = useState(0);
  return (
    <>
      <Child name="John" />
      <Button title="Increment" onPress={() => setCount(count + 1)} />
    </>
  );
};
```

## React Hooks Best Practices

### Hook Rules

Call hooks at the top level of functional components or custom hooks. Never call hooks inside loops, conditions, or nested functions.

**Incorrect (Hook in Conditional):**
```typescript
function MyComponent() {
  if (true) {
    const [count, setCount] = useState(0); // ❌ Violates rules
  }
  return <Text>{count}</Text>;
}
```

**Correct:**
```typescript
function MyComponent() {
  const [count, setCount] = useState(0);
  if (true) {
    // Logic here is safe
  }
  return <Text>{count}</Text>;
}
```

### useState

**When to Use**: For managing local component state (form inputs, UI toggles, counters).

**Functional Updates**: Use functional updates when new state depends on previous state to avoid stale state.

**Incorrect (Stale State Risk):**
```typescript
const [count, setCount] = useState(0);
const increment = () => {
  setCount(count + 1); // ❌ May cause stale state
};
```

**Correct (Functional Update):**
```typescript
const [count, setCount] = useState(0);
const increment = () => {
  setCount((prevCount) => prevCount + 1);
};
```

### useEffect

**When to Use**: For side effects (API calls, subscriptions, timers, subscriptions).

**Dependency Array**: Always provide a dependency array. Use empty array `[]` for mount-only effects. Include all variables used in the effect.

**Incorrect (Runs Every Render):**
```typescript
useEffect(() => {
  console.log("Component mounted");
  // Missing dependency array = runs every render
});
```

**Correct (Runs Only on Mount):**
```typescript
useEffect(() => {
  console.log("Component mounted");
}, []); // Empty array = mount only
```

### useEffect Cleanup

Always return a cleanup function for subscriptions, timers, or event listeners.

**Incorrect (Memory Leak):**
```typescript
useEffect(() => {
  setInterval(() => {
    console.log("Running...");
  }, 1000); // ❌ No cleanup
}, []);
```

**Correct (Cleanup Function):**
```typescript
useEffect(() => {
  const interval = setInterval(() => {
    console.log("Running...");
  }, 1000);
  return () => clearInterval(interval); // Cleanup on unmount
}, []);
```

### Conditional Logic in Effects

Place conditional logic inside the effect, not around the hook call.

**Incorrect (Hook in Conditional):**
```typescript
if (someCondition) {
  useEffect(() => {
    console.log("This will break the rules");
  }, []);
}
```

**Correct (Conditional Inside Effect):**
```typescript
useEffect(() => {
  if (someCondition) {
    console.log("This is safe");
  }
}, [someCondition]);
```

### useCallback

**When to Use**: For memoizing functions passed as props or used in dependency arrays of other hooks.

**Why**: Prevents unnecessary child re-renders and infinite effect loops.

**Incorrect (Function Created Every Render):**
```typescript
const MyComponent: React.FC<{ a: number; b: number }> = ({ a, b }) => {
  const onPress = () => {
    // Function recreated every render
  };
  return <ChildComponent onPress={onPress} />;
};
```

**Correct (Memoized Function):**
```typescript
const MyComponent: React.FC<{ a: number; b: number }> = ({ a, b }) => {
  const onPress = useCallback(() => {
    // Function reused if a & b don't change
  }, [a, b]);
  return <ChildComponent onPress={onPress} />;
};
```

### useMemo

**When to Use**: For memoizing expensive computations or objects/arrays passed to child components.

**Incorrect (Computation Every Render):**
```typescript
const MyComponent: React.FC<{ items: number[] }> = ({ items }) => {
  const sortedItems = items.sort((a, b) => a - b); // ❌ Runs every render
  return <Text>{sortedItems.join(", ")}</Text>;
};
```

**Correct (Memoized Computation):**
```typescript
const MyComponent: React.FC<{ items: number[] }> = ({ items }) => {
  const sortedItems = useMemo(() => {
    return [...items].sort((a, b) => a - b);
  }, [items]);
  return <Text>{sortedItems.join(", ")}</Text>;
};
```

### useRef

**When to Use**: For mutable references that persist across renders without triggering re-renders (e.g., TextInput focus, timer IDs, storing mutable values).

**Important**: Access/modify via `.current`. Do not use refs for state that affects rendering.

**Incorrect (Hook in Conditional):**
```typescript
const MyComponent: React.FC = () => {
  if (true) {
    const inputRef = useRef<TextInput | null>(null); // ❌ Hook in conditional
  }
  return <TextInput />;
};
```

**Correct (Focus Input with Ref):**
```typescript
const MyComponent: React.FC = () => {
  const inputRef = useRef<TextInput | null>(null);
  const focusInput = () => inputRef.current?.focus();
  return (
    <View>
      <TextInput ref={inputRef} />
      <Button title="Focus Input" onPress={focusInput} />
    </View>
  );
};
```

### useLayoutEffect

**When to Use**: For side effects that need to run synchronously after DOM updates but before painting (e.g., measuring layout, avoiding visual flicker). Rarely used in React Native.

**Rule**: Always provide a dependency array.

**Incorrect (No Dependency Array):**
```typescript
const MyComponent: React.FC = () => {
  useLayoutEffect(() => {
    console.log('Runs every render'); // ❌ No dependency array
  });
  return <View><Text>App</Text></View>;
};
```

**Correct (With Dependency Array):**
```typescript
const MyComponent: React.FC = () => {
  const viewRef = useRef<View | null>(null);
  useLayoutEffect(() => {
    console.log('Measure layout');
    return () => console.log('Cleanup');
  }, []);
  return <View ref={viewRef}><Text>App</Text></View>;
};
```

### Custom Hooks

**When to Use**: To extract reusable logic used across multiple components (screen dimensions, network status, device sensors, app lifecycle tracking).

**Example: useOnlineStatus**
```typescript
function useOnlineStatus(): boolean {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    NetInfo.addEventListener("connectionChange", (state) => {
      setIsOnline(state.isConnected ?? false);
    });
    return () => {
      // Cleanup
    };
  }, []);
  return isOnline;
}

// Usage
function MyComponent() {
  const isOnline = useOnlineStatus();
  return <Text>{isOnline ? "Online" : "Offline"}</Text>;
}
```

**Example: useDisableBackButton**
```typescript
function useDisableBackButton(): void {
  useEffect(() => {
    const handleBackPress = () => true; // Block back button
    BackHandler.addEventListener('hardwareBackPress', handleBackPress);
    return () => BackHandler.removeEventListener('hardwareBackPress', handleBackPress);
  }, []);
}

// Usage
function MyComponent() {
  useDisableBackButton();
  return <View><Text>Back button disabled</Text></View>;
}
```

**Example: useAppState**
```typescript
function useAppState(): AppStateStatus {
  const [appState, setAppState] = useState<AppStateStatus>(AppState.currentState);
  useEffect(() => {
    const handleAppStateChange = (nextState: AppStateStatus) => setAppState(nextState);
    const subscription = AppState.addEventListener('change', handleAppStateChange);
    return () => subscription.remove();
  }, []);
  return appState;
}

// Usage
function MyComponent() {
  const appState = useAppState();
  return <View><Text>App State: {appState}</Text></View>;
}
```

## Redux Selectors

### Custom Selectors

Use custom selectors for complex state selection or code reused in multiple places.

**Keyed Loader Selector:**
```typescript
const defaultValue = { loading: false };
export const loaderSelector = (name: string) => (store: RootState) => {
  return store.loader.loaders[name] || defaultValue;
};

// Usage
const { loading } = useSelector(loaderSelector("AsynchronousTask"));
```

### Memoized Selectors with reselect

Use memoized selectors from `reselect` to improve performance and avoid unnecessary re-renders.

```typescript
import { createSelector } from "reselect";

const selectUsers = (state: RootState) => state.users;
export const getUserById = createSelector(
  [selectUsers, (_, userId: string) => userId],
  (users, userId) => users[userId]
);

// Usage
const user = useSelector((state: RootState) => getUserById(state, userId));
```

## State Management Best Practices

### Avoid Unnecessary State

If you can derive a value from props or state, avoid storing it separately.

**Incorrect (Unnecessary State):**
```typescript
const [totalPrice, setTotalPrice] = useState(0);
useEffect(() => {
  setTotalPrice(items.reduce((sum, item) => sum + item.price, 0)); // ❌ Redundant
}, [items]);
```

**Correct (Derived Value):**
```typescript
const totalPrice = items.reduce((sum, item) => sum + item.price, 0);
```

### Use Context for Global State Instead of Prop Drilling

If a state is needed across multiple components, use Context API instead of passing props through many levels.

```typescript
const ThemeContext = createContext<string>("light");

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <ChildComponent />
    </ThemeContext.Provider>
  );
}

function ChildComponent() {
  const theme = useContext(ThemeContext);
  return <Text>Theme: {theme}</Text>;
}
```

## Testing & Code Coverage

### Testing Overview

**What Is Testing?** Testing is a line-by-line review of how your code will execute. A suite of tests comprises various pieces of code to verify whether an application executes successfully and without error.

**Why Test?**
- **Prevent Regression**: Avoid reappearance of previously fixed bugs
- **Ensure Functionality**: Verify complex components work as intended
- **Improve Robustness**: Make apps more robust and less prone to error
- **Verify Intent**: Confirm that code does what you expect

### Testing Libraries

KeylessRN uses:
- **@testing-library/react-native** - Component testing and DOM queries
- **axios-mock-adapter** - Mocking HTTP requests
- **jest** - Test runner and assertion library

### Types of Testing

**Unit Test**
Tests individual functions, methods, or components in isolation. Verifies that each unit performs as expected.
```typescript
// Example: Testing a utility function
test('adds two numbers correctly', () => {
  const result = add(2, 3);
  expect(result).toBe(5);
});
```

**Component Test**
Tests individual components in isolation from others. Verifies component functionality, props handling, and event firing.
```typescript
test('Button component calls onPress when tapped', () => {
  const onPress = jest.fn();
  const { getByTestId } = render(
    <Button title="Click" onPress={onPress} testID="btn" />
  );
  fireEvent.press(getByTestId('btn'));
  expect(onPress).toHaveBeenCalled();
});
```

**Snapshot Test**
Captures component code at a moment in time to detect unexpected UI changes.
```typescript
test('Component snapshot matches', () => {
  const { toJSON } = render(<MyComponent />);
  expect(toJSON()).toMatchSnapshot();
});
```

### Key Testing Concepts

**Mock**
A mock is a convincing duplicate of an object or module without real inner workings. Mocks reduce dependencies and speed up test execution.

```typescript
// Mock entire module
jest.mock("@react-navigation/native", () => {
  return {
    useNavigation: () => ({})
  }
});

// Mock with actual module functionality
jest.mock("@react-navigation/native", () => {
  const actualNav = jest.requireActual("@react-navigation/native");
  return {
    ...actualNav,
    useNavigation: () => ({})
  }
});
```

**Spy**
Tracks function calls while preserving behavior. Verifies whether functions are called correctly.

```typescript
import * as navigation from "@react-navigation/native";

let spy;
beforeEach(() => {
  jest.resetAllMocks();
  spy = jest.spyOn(navigation, "useNavigation").mockImplementation(() => ({}));
});

test('useNavigation is called', () => {
  // ... component that uses useNavigation
  expect(spy).toHaveBeenCalled();
  expect(spy.mock.calls.length).toBe(1);
});
```

**Describe Blocks**
Organize related tests in logical groups.

```typescript
describe('LoginWithEmail', () => {
  describe('Resident Login', () => {
    test('login succeeds with valid credentials', () => {});
    test('login fails with invalid credentials', () => {});
  });
  
  describe('Agent Login', () => {
    test('agent login succeeds', () => {});
  });
});
```

**Matchers**
Create assertions to compare expected vs actual values.

```typescript
expect(value).toBe(5);                    // Strict equality
expect(value).toEqual({ name: 'John' }); // Deep equality
expect(spy).toHaveBeenCalled();          // Function called
expect(spy).toHaveBeenCalledTimes(2);   // Called exactly N times
expect(array).toHaveLength(3);           // Array length
expect(component).toMatchSnapshot();     // Snapshot match
```

**Note**: Use `toEqual()` for objects, `toBe()` for primitives. `toBe()` checks object reference; `toEqual()` checks value.

### Testing React Native Components

**Component Testing Example:**

```typescript
import { fireEvent, render } from "@testing-library/react-native";
import { AnchorButton } from './AnchorButton';

describe('AnchorButton', () => {
  test('button click calls onPress callback', () => {
    const onPress = jest.fn();
    const { getByTestId } = render(
      <AnchorButton
        testID="anchorButton"
        title="Click Me"
        onPress={onPress}
        disabled={false}
      />
    );
    
    const button = getByTestId('anchorButton');
    fireEvent.press(button);
    
    expect(onPress).toHaveBeenCalledTimes(1);
    expect(button.props.disabled).toBe(false);
  });
});
```

**Snapshot Testing Example:**

```typescript
describe('AnchorButton', () => {
  test('snapshot test', () => {
    const onPress = jest.fn();
    const { toJSON } = render(
      <AnchorButton
        testID="anchorButton"
        title="Click Me"
        onPress={onPress}
        disabled={false}
      />
    );
    
    expect(toJSON()).toMatchSnapshot();
  });
});
```

Snapshots capture component output and flag unexpected changes. Run `jest -u` to update snapshots after intentional changes.

### Mocking API Calls

Use `axios-mock-adapter` to mock HTTP requests without making real API calls.

```typescript
import MockAdapter from 'axios-mock-adapter';
import { authAxios, mainAxios } from './apiWrapper';

// Create mock adapters for each axios instance
const mockAuthAxios = new MockAdapter(authAxios);
const mockMainAxios = new MockAdapter(mainAxios);

// Mock POST /oauth/token
mockAuthAxios.onPost('/oauth/token').reply(200, {
  success: true,
  access_token: 'token123',
  refresh_token: 'refresh123',
  role: 'Resident'
});

// Mock GET /api/occupants/profile
mockMainAxios.onGet('/api/occupants/profile').reply(200, {
  success: true,
  profileDetails: {
    email: 'user@example.com',
    id: '123'
  }
});
```

### Full Integration Test Example

**Resident Login Test:**

```typescript
import React from 'react';
import { act, fireEvent, render } from '@testing-library/react-native';
import { LoginWithEmail } from '../Login/LoginWithEmail';
import { Provider } from 'react-redux';
import { store } from '../../common/store/mockStore';
import { Alert } from 'react-native';
import { user } from '../../common/user';
import { mockApi } from '../../common/apiWrapper/mockApi';
import * as SecureStorage from '../../common/secureStorage';
import * as actions from '../../common/store/typeSafe';

describe('LoginWithEmail Screen Test', () => {
  let dispatchedActions = [];
  
  beforeEach(() => {
    dispatchedActions = [];
    jest.resetAllMocks();
    
    // Mock secure storage
    jest.spyOn(SecureStorage, 'setSecureData').mockImplementation(() => {});
    
    // Mock user login
    jest.spyOn(user, 'login').mockImplementation(() => {});
    
    // Mock alert
    jest.spyOn(Alert, 'alert').mockImplementation(() => {});
    
    // Spy on action dispatcher to track all dispatched actions
    jest.spyOn(actions, 'action').mockImplementation((type: string, payload: any) => {
      dispatchedActions.push({ type, payload });
      return { type, payload };
    });
    
    // Mock login API response
    mockApi.onPost(true, '/oauth/token', 200, {
      success: true,
      access_token: 'access_token_123',
      refresh_token: 'refresh_token_123',
      id_token: 'id_token_123',
      role: 'Resident'
    });
    
    // Mock profile API response
    mockApi.onGet(false, '/api/occupants/profile', 200, {
      success: true,
      profileDetails: {
        email: 'residentlogin@mail.com'
      }
    });
  });
  
  test('Resident Login Success', async () => {
    // Render component wrapped with Redux Provider
    const { getByTestId, getByText } = render(
      <Provider store={store}>
        <LoginWithEmail
          type="resident"
          email=""
          password=""
          showLoginWithFaceIdTouchIDUI={jest.fn()}
        />
      </Provider>
    );
    
    // Verify email field exists
    getByText('EMAIL');
    const email = getByTestId('email');
    
    // Change email field (Formik is async, wrap with act)
    await act(async () => {
      fireEvent.changeText(email, 'residentlogin@gmail.com');
    });
    expect(email.props.value).toBe('residentlogin@gmail.com');
    
    // Verify password field exists
    getByText('PASSWORD');
    const password = getByTestId('password');
    
    // Change password field
    await act(async () => {
      fireEvent.changeText(password, 'residentLogin');
    });
    expect(password.props.value).toBe('residentLogin');
    
    // Verify login button is enabled
    getByText('LOGIN');
    const login = getByTestId('login');
    expect(login.props.disabled).toBe(false);
    
    // Press login button
    await act(async () => {
      fireEvent.press(login);
    });
    
    // Verify all expected actions were dispatched
    expect(dispatchedActions).toMatchSnapshot();
  });
});
```

### Testing Best Practices

**1. Reset Mocks Between Tests**
```typescript
beforeEach(() => {
  jest.resetAllMocks(); // Clear mock call history
});
```

**2. Use testID for Element Selection**
```typescript
// Good
const button = getByTestId('submit-button');

// Avoid
const button = getByText('Submit'); // Fragile, breaks on text change
```

**3. Wrap Async Operations with act()**
```typescript
// Required for async state updates, Formik changes, timers
await act(async () => {
  fireEvent.changeText(input, 'new value');
});
```

**4. Mock External Dependencies**
```typescript
// Mock navigation
jest.mock('@react-navigation/native', () => ({
  useNavigation: jest.fn()
}));

// Mock API
mockApi.onGet('/api/endpoint').reply(200, { data: 'mock' });

// Mock system modules
jest.mock('react-native/Libraries/Alert/Alert', () => ({
  alert: jest.fn()
}));
```

**5. Test User Interactions, Not Implementation**
```typescript
// Good: Test what user sees and does
test('user can login with email and password', () => {
  const { getByTestId } = render(<LoginScreen />);
  fireEvent.changeText(getByTestId('email'), 'user@example.com');
  fireEvent.changeText(getByTestId('password'), 'password123');
  fireEvent.press(getByTestId('login-button'));
  expect(mockNavigation.navigate).toHaveBeenCalledWith('Home');
});

// Avoid: Testing implementation details
test('LoginScreen calls loginSaga', () => {
  // Don't test saga internals; test observable behavior
});
```

**6. Keep Tests Isolated**
```typescript
// Each test should be independent
// Don't rely on test execution order
// Reset state/mocks before each test
beforeEach(() => {
  jest.resetAllMocks();
  // Recreate fixtures
});
```

**7. Use Descriptive Test Names**
```typescript
// Good
test('should submit form with valid email and password', () => {});
test('should show error message when API fails', () => {});
test('should disable submit button while loading', () => {});

// Avoid
test('form works', () => {});
test('login 1', () => {});
```

### Running Tests

```bash
# Run all tests
yarn test

# Run tests in watch mode
yarn test --watch

# Run tests for specific file
yarn test LoginWithEmail.test.tsx

# Update snapshots
yarn test -u

# Run with coverage report
yarn test --coverage

# Run specific test by name
yarn test -t "Resident Login Success"
```

### Testing Setup

- **Jest Config**: Configured in `jest.config.js` with React Native preset
- **Mock Store**: [`src/common/store/mockStore.ts`](src/common/store/mockStore.ts) - Redux store for testing
- **Test Utilities**: [`jest/util.js`](jest/util.js) - `customRender` wraps components with Redux Provider
- **Mock API**: [`src/common/apiWrapper/mockApi.ts`](src/common/apiWrapper/mockApi.ts) - Axios mock setup

**Example Test File Structure:**

```typescript
// Setup and imports
import { render, fireEvent, act } from 'jest/util';
import { store } from '../common/store/mockStore';
import { mockApi } from '../common/apiWrapper/mockApi';

// Group related tests
describe('Feature or Component Name', () => {
  // Setup before each test
  beforeEach(() => {
    jest.resetAllMocks();
    // Setup mocks, spies, fixtures
  });
  
  // Individual test case
  test('should do something', async () => {
    // Arrange: Set up test data
    // Act: Perform user action
    // Assert: Verify result
  });
});
```

## Critical Build & Test Commands

```bash
# Run/Build (iOS examples - Android similar)
yarn runIOSKeylessFlavour          # Run default flavor
yarn runIOSProgressFlavour         # Run progress flavor
yarn runIOSNHRFlavour             # Run NHR flavor

# Testing & Linting
yarn test                          # Jest tests (JEST_TEST_RUNNING=true env set)
yarn lint                          # ESLint check
yarn lint-fix                      # Auto-fix lint
yarn type-check                    # TypeScript check
yarn lint-flavours                 # Lint across all flavor configs

# iOS-specific
cd ios && pod install              # After package.json changes
yarn cleanIOS                      # Clear build cache

# Android-specific
yarn cleanAndroid                  # Clear Gradle cache
yarn build-android                 # Release build
```

## State Structure (What You'll Interact With)

Redux store shape (simplified):
```typescript
{
  login: { loggedIn: boolean },
  app: { userRole: 'Resident' | 'Manager' | 'InstallerV3', ... },
  home: { 
    properties, selectedAsset, invitations, 
    freeDevices, freeHubs, selectedAccount, 
    remoteConfig, communities, timers 
  },
  loader: { loading: boolean, loaders: { [key]: {loading, message, error} } },
  connectivity: { online: boolean },
  hubSearch: { foundHubs, isScanning, ... },
  locks: { lockDetails, testDetails, ... },
  remot: { invitations, locks, ... },
  intercom: { hmsInstance, chat, callHistory, ... }
}
```

**Always persist state via:**
```typescript
// Whitelisted for AsyncStorage persistence
whitelist: ['app', 'home', 'login', 'userGuide', 'faceIdReducer', 'shortRangeOfflineActivity', 'dashboard']
```

## Feature-Specific Patterns

### Device/Lock Management
- **Selectors**: Use keyed loaders for async operations. Example: `loaderSelector('TestLog')` returns `{ loading, message, error }`
- **Test Flow**: Complex state in `state.hubSearch.testDetails` tracks test cases, MQTT connectivity, device categories
- **MQTT Integration**: Real-time device events via MQTTManager; sagas subscribe/unsubscribe on component mount/unmount

### Role-Based UI
- **Helpers**: [`src/common/UserManagement/roles.ts`](src/common/UserManagement/roles.ts) exports: `isResident()`, `isManager()`, `isInstaller()`, `isV3Installer()`
- **Usage**: Wrap navigation/features conditionally on role checks
- **Persistence**: userRole stored in Redux, synced on auth

### Offline Support
- **MMKV Storage**: Fast key-value store in [`src/common/MMKV/`](src/common/MMKV/) for critical data
- **Connectivity Check**: `state.connectivity.online` toggle saga effects (sagas pause offline, resume online)
- **Pattern**: Check online before API calls; queue offline, sync on reconnect

## Navigation & Screen Structure

- **Navigation**: React Navigation (Stack/Tab/Drawer)
- **Main entry**: [`src/Home/index.tsx`](src/Home/index.tsx) - large navigator setup file
- **Pattern**: Screens dispatch Redux actions → sagas handle side effects → reducers update state → components re-render

Example navigation flow:
```typescript
// In screen component
const navigation = useNavigation();
const dispatch = useDispatch();
const state = useSelector((state: RootState) => state.home);

const handleAction = () => {
  dispatch(fetchPropertyAction());  // Trigger saga
  navigation.navigate('DetailScreen');
};
```

## Common Developer Mistakes to Avoid

1. **Modifying state directly**: Always use reducers (immer-based createReducer ensures immutability)
2. **Missing dependency arrays in sagas**: `takeEvery(ACTION, sagaFn)` keeps saga alive
3. **Forgetting to unsubscribe**: MQTT & event listeners must clean up in saga `finally` blocks
4. **Not handling offline**: Check `state.connectivity.online` before API calls in sagas
5. **Persisting transient state**: Only whitelist reducers meant to survive app reload
6. **Not using loaders correctly**: Always wrap async operations with `startLoadingAction` → success/failed → dispatch result

## Reusable Components from src/common/components

Always check for existing components before creating new ones. All commonly-used UI components are exported from [`src/common/components/index.ts`](src/common/components/index.ts).

**Import components:**
```typescript
import {
  CurvedButton,
  FTextInputWithLabel,
  Card,
  Label,
  LoadingIndicator,
  Dropdown,
  CheckButton,
  Modal,
  DateTimePicker,
  PhoneNumber,
  RadioButton,
  PasswordField,
  Divider,
  Switch,
  ValidationErrorMessage
} from '../../../common/components';
```

### Core UI Components

**Buttons & Controls:**
- `CurvedButton` - Primary styled button with loading animation, icon support, size variants (xs, s, m, l, xl, xxl), inverse mode
- `CheckButton` - Checkbox component with label and custom styling
- `RadioButton` - Radio button for option selection
- `AnchorButton` - Link-style button
- `SwipeButton` - Swipeable action button
- `ToggleButton` - Toggle switch button

**Input Components:**
- `FTextInputWithLabel` - Text input with label, error message, and validation state
- `PasswordField` - Password input with show/hide toggle
- `PhoneNumber` - Phone number input with country code selector
- `OtpInputs` - OTP/PIN code input
- `Dropdown` - Picker-based dropdown (iOS/Android native)
- `CustomDropdown` - Scrollable dropdown with search/pagination

**Display Components:**
- `Label` - Text with typography variants (primary, secondary, bold, xl, l, m, s)
- `Card` - Container with shadow and rounded corners
- `Divider` - Visual separator line
- `Modal` - Dialog component with customizable content
- `LoadingIndicator` - Spinner/loader animation
- `DotAnimation` - Animated dot indicator

**Specialized Components:**
- `DateTimePicker` - Date and time selection with modal
- `Switch` - Toggle switch with label (via `SwitchButtonWithLabel`)
- `NumberBadge` - Badge with count/number display
- `DeviceListCard` - Device list item with details
- `DeviceUnreachableBanner` - Status indicator for unreachable devices

### Component Usage Examples

**Form with multiple inputs:**
```typescript
import { CurvedButton, FTextInputWithLabel, Label, PasswordField, ValidationErrorMessage } from '../../../common/components';
import { Formik } from 'formik';
import * as Yup from 'yup';

const loginSchema = Yup.object().shape({
  email: Yup.string().email().required(),
  password: Yup.string().min(6).required()
});

<Formik
  initialValues={{ email: '', password: '' }}
  validationSchema={loginSchema}
  onSubmit={(values) => dispatch(loginAction(values))}
>
  {({ values, errors, touched, handleChange, handleBlur, handleSubmit }) => (
    <>
      <FTextInputWithLabel
        label="Email"
        value={values.email}
        onChangeText={handleChange('email')}
        onBlur={handleBlur('email')}
        error={touched.email && errors.email}
      />
      <PasswordField
        label="Password"
        value={values.password}
        onChangeText={handleChange('password')}
        onBlur={handleBlur('password')}
        error={touched.password && errors.password}
      />
      <CurvedButton
        title="Login"
        onPress={handleSubmit}
        disableButton={!isValid}
      />
    </>
  )}
</Formik>
```

**Dropdown selection:**
```typescript
import { Dropdown, Label } from '../../../common/components';

<Dropdown
  value={selectedValue}
  data={[
    { label: 'Option 1', value: 'opt1' },
    { label: 'Option 2', value: 'opt2' },
    { label: 'Option 3', value: 'opt3' }
  ]}
  onValueChange={(value) => setSelectedValue(value)}
  placeholder={{ label: 'Select an option' }}
/>
```

**Card with button:**
```typescript
import { Card, Label, CurvedButton } from '../../../common/components';

<Card>
  <Label title="Device Status" xl bold />
  <Label secondary title="Connected" />
  <CurvedButton
    title="Configure"
    onPress={() => navigation.navigate('Config')}
  />
</Card>
```

**Modal with form:**
```typescript
import { Modal, FTextInputWithLabel, CurvedButton } from '../../../common/components';

<Modal
  visibility={isModalVisible}
  onBackdropPress={() => setModalVisible(false)}
>
  <FTextInputWithLabel
    label="Enter code"
    value={code}
    onChangeText={setCode}
  />
  <CurvedButton
    title="Submit"
    onPress={handleSubmit}
  />
</Modal>
```

### Component Props Patterns

Most components follow consistent prop patterns:

```typescript
// Common props across components
{
  // Styling
  style?: ViewStyle | TextStyle,
  buttonStyle?: ViewStyle,
  labelStyle?: TextStyle,
  viewStyle?: ViewStyle,
  
  // State
  value?: any,
  error?: string | boolean,
  disabled?: boolean,
  
  // Events
  onPress?: () => void,
  onChangeText?: (text: string) => void,
  onChange?: (value: any) => void,
  onBlur?: () => void,
  
  // Accessibility
  testID?: string,
  accessibilityLabel?: string,
  
  // Variants
  inverse?: boolean,
  bold?: boolean,
  primary?: boolean,
  secondary?: boolean,
  xs?: boolean, // CurvedButton size variants
  s?: boolean,
  m?: boolean,
  l?: boolean,
  xl?: boolean
}
```

### When to Create New Components

❌ **Don't create new components if:**
- A similar reusable component exists
- It's just a wrapper around an existing component
- It's specific to one feature (use in that feature folder instead)

✅ **Do create new components if:**
- The component is used across 3+ different features
- It's a complex, self-contained UI pattern
- It wraps multiple components with significant shared logic
- Place in [`src/common/components/`](src/common/components/) with proper exports

### Detailed Props Documentation

**CurvedButton Props:**
```typescript
interface CurvedButtonProps {
  title: string;                           // Button text
  onPress?: () => void;                    // Click handler
  disableButton?: boolean;                 // Disable state
  inverse?: boolean;                       // Inverse style (secondary appearance)
  buttonStyle?: ViewStyle;                 // Custom button style
  textStyle?: TextStyle;                   // Custom text style
  Icon?: React.ReactNode;                  // Left icon
  xs?: boolean; s?: boolean; m?: boolean; l?: boolean; xl?: boolean; xxl?: boolean; // Size variants
  isAnimating?: boolean;                   // Show loading animation
  showCircularLoadingIndicator?: boolean;  // Circular progress on button
  testID?: string;                         // Testing identifier
  accessibilityLabel?: string;             // Accessibility label
}

// Usage examples
<CurvedButton title="Save" onPress={handleSave} />
<CurvedButton title="Loading..." isAnimating={true} disableButton={true} />
<CurvedButton title="Delete" inverse xl Icon={<TrashIcon />} onPress={handleDelete} />
```

**FTextInputWithLabel Props:**
```typescript
interface FTextInputWithLabelProps {
  label?: string;                    // Input label
  value: string;                     // Current value
  onChangeText: (text: string) => void; // Change handler
  error?: string | boolean;          // Error message or boolean
  placeholder?: string;              // Placeholder text
  secureTextEntry?: boolean;         // Hide text (for passwords)
  multiline?: boolean;               // Multi-line input
  keyboardType?: KeyboardTypeOptions; // Keyboard type
  editable?: boolean;                // Editable state
  viewStyle?: ViewStyle;             // Container style
  labelStyle?: TextStyle;            // Label style
  textInputStyle?: TextStyle;        // Input style
  testID?: string;
  accessibilityLabel?: string;
  onFocus?: () => void;
  onBlur?: () => void;
}

// Usage with validation
<FTextInputWithLabel
  label="Email"
  value={email}
  onChangeText={setEmail}
  error={touched.email && errors.email}
  keyboardType="email-address"
  placeholder="your@email.com"
/>
```

**Dropdown Props:**
```typescript
interface DropdownProps {
  value: string | number;
  data: Array<{ label: string; value: string | number }>;
  onValueChange: (value: any) => void;
  placeholder?: { label: string; value: string | number };
  disabled?: boolean;
  testID?: string;
  accessibilityLabel?: string;
  outerViewStyle?: ViewStyle;
}

// Usage
<Dropdown
  value={selectedRole}
  data={[
    { label: 'Resident', value: 'resident' },
    { label: 'Manager', value: 'manager' },
    { label: 'Installer', value: 'installer' }
  ]}
  onValueChange={setSelectedRole}
  placeholder={{ label: 'Select role', value: '' }}
/>
```

**DateTimePicker Props:**
```typescript
interface DateTimePickerProps {
  label: string;
  value: string;                     // ISO format date string
  date: Date;                        // JavaScript Date object
  onConfirm: (date: Date) => void;   // Date selected handler
  minimumDate?: Date;                // Minimum selectable date
  maximumDate?: Date;                // Maximum selectable date
  textInputStyle?: TextStyle;
  viewStyle?: ViewStyle;
  testID?: string;
}

// Usage
<DateTimePicker
  label="Tour Date"
  value={Utility.formatDateToString(tourDate)}
  date={tourDate}
  onConfirm={(date) => setTourDate(date)}
  minimumDate={new Date()}
/>
```

### Advanced Patterns

**Composition with Multiple Components:**
```typescript
// Create specialized components by composing simpler ones
const LoginForm: React.FC<LoginFormProps> = ({ onSubmit, loading }) => {
  return (
    <Formik
      initialValues={{ email: '', password: '' }}
      validationSchema={Yup.object().shape({
        email: Yup.string().email().required(),
        password: Yup.string().min(8).required()
      })}
      onSubmit={onSubmit}
    >
      {({ values, errors, touched, handleChange, handleBlur, handleSubmit }) => (
        <ScrollView contentContainerStyle={styles.container}>
          <FTextInputWithLabel
            label="Email Address"
            value={values.email}
            onChangeText={handleChange('email')}
            onBlur={handleBlur('email')}
            error={touched.email && errors.email}
            keyboardType="email-address"
          />
          
          <PasswordField
            label="Password"
            value={values.password}
            onChangeText={handleChange('password')}
            onBlur={handleBlur('password')}
            error={touched.password && errors.password}
          />
          
          <Divider />
          
          {loading && <LoadingIndicator />}
          
          <CurvedButton
            title="Sign In"
            onPress={handleSubmit}
            disableButton={loading || !isValid}
            isAnimating={loading}
          />
        </ScrollView>
      )}
    </Formik>
  );
};
```

**Custom Hook with Components:**
```typescript
// Use custom hooks to manage component state and logic
const useForm = (initialValues, onSubmit) => {
  const [values, setValues] = React.useState(initialValues);
  const [errors, setErrors] = React.useState({});
  const [touched, setTouched] = React.useState({});
  
  const handleChange = (field) => (text) => {
    setValues(prev => ({ ...prev, [field]: text }));
  };
  
  const handleBlur = (field) => () => {
    setTouched(prev => ({ ...prev, [field]: true }));
  };
  
  const handleSubmit = async () => {
    const newErrors = validateForm(values);
    setErrors(newErrors);
    
    if (Object.keys(newErrors).length === 0) {
      await onSubmit(values);
    }
  };
  
  return { values, errors, touched, handleChange, handleBlur, handleSubmit };
};

// Usage in component
function DeviceForm() {
  const { values, errors, touched, handleChange, handleBlur, handleSubmit } = 
    useForm({ name: '', category: '' }, onFormSubmit);
  
  return (
    <>
      <FTextInputWithLabel
        label="Device Name"
        value={values.name}
        onChangeText={handleChange('name')}
        onBlur={handleBlur('name')}
        error={touched.name && errors.name}
      />
      
      <Dropdown
        value={values.category}
        data={deviceCategories}
        onValueChange={handleChange('category')}
      />
      
      <CurvedButton title="Save" onPress={handleSubmit} />
    </>
  );
}
```

**Conditional Rendering with Components:**
```typescript
// Show different UI based on state or role
const DeviceCard: React.FC<DeviceCardProps> = ({ device, userRole }) => {
  const isManager = isManagerRole(userRole);
  const isOnline = device.status === 'online';
  
  return (
    <Card>
      <Label title={device.name} bold xl />
      
      {!isOnline && <DeviceUnreachableBanner />}
      
      {isManager && (
        <CurvedButton
          title="Configure"
          onPress={() => navigation.navigate('DeviceSettings')}
        />
      )}
      
      {isOnline && (
        <CurvedButton
          title="Control"
          onPress={() => openDeviceControl(device.id)}
        />
      )}
    </Card>
  );
};
```

### Component Testing Examples

**Testing Input Components:**
```typescript
import { render, fireEvent } from 'jest/util';
import { FTextInputWithLabel } from '../FTextInputWithLabel';

describe('FTextInputWithLabel', () => {
  it('renders label and input field', () => {
    const { getByText, getByDisplayValue } = render(
      <FTextInputWithLabel
        label="Email"
        value="test@example.com"
        onChangeText={jest.fn()}
      />
    );
    
    expect(getByText('Email')).toBeTruthy();
    expect(getByDisplayValue('test@example.com')).toBeTruthy();
  });
  
  it('displays error message when provided', () => {
    const { getByText } = render(
      <FTextInputWithLabel
        label="Email"
        value=""
        onChangeText={jest.fn()}
        error="Email is required"
      />
    );
    
    expect(getByText('Email is required')).toBeTruthy();
  });
  
  it('calls onChangeText when user types', () => {
    const mockOnChange = jest.fn();
    const { getByDisplayValue } = render(
      <FTextInputWithLabel
        label="Email"
        value=""
        onChangeText={mockOnChange}
      />
    );
    
    fireEvent.changeText(getByDisplayValue(''), 'new@email.com');
    expect(mockOnChange).toHaveBeenCalledWith('new@email.com');
  });
});
```

**Testing Button Components:**
```typescript
import { render, fireEvent } from 'jest/util';
import { CurvedButton } from '../CurvedButton';

describe('CurvedButton', () => {
  it('renders button with title', () => {
    const { getByText } = render(
      <CurvedButton title="Click Me" onPress={jest.fn()} />
    );
    
    expect(getByText('Click Me')).toBeTruthy();
  });
  
  it('calls onPress when clicked', () => {
    const mockOnPress = jest.fn();
    const { getByRole } = render(
      <CurvedButton title="Click" onPress={mockOnPress} />
    );
    
    fireEvent.press(getByRole('button'));
    expect(mockOnPress).toHaveBeenCalled();
  });
  
  it('disables button when disableButton is true', () => {
    const mockOnPress = jest.fn();
    const { getByRole } = render(
      <CurvedButton
        title="Click"
        onPress={mockOnPress}
        disableButton={true}
      />
    );
    
    fireEvent.press(getByRole('button'));
    expect(mockOnPress).not.toHaveBeenCalled();
  });
  
  it('shows loading indicator when animating', () => {
    const { getByTestId } = render(
      <CurvedButton
        title="Saving"
        onPress={jest.fn()}
        isAnimating={true}
        testID="save-button"
      />
    );
    
    expect(getByTestId('save-button')).toBeTruthy();
  });
});
```

**Testing Modal Components:**
```typescript
import { render, fireEvent } from 'jest/util';
import { Modal } from '../Modal';

describe('Modal', () => {
  it('renders modal when visibility is true', () => {
    const { getByText } = render(
      <Modal visibility={true}>
        <Label title="Modal Content" />
      </Modal>
    );
    
    expect(getByText('Modal Content')).toBeTruthy();
  });
  
  it('calls onBackdropPress when backdrop is pressed', () => {
    const mockOnBackdropPress = jest.fn();
    const { getByTestId } = render(
      <Modal
        visibility={true}
        onBackdropPress={mockOnBackdropPress}
        testID="modal-backdrop"
      >
        <Label title="Content" />
      </Modal>
    );
    
    fireEvent.press(getByTestId('modal-backdrop'));
    expect(mockOnBackdropPress).toHaveBeenCalled();
  });
});
```

### Theme & Styling Integration

**Using Theme Colors:**
```typescript
// Access theme in component
import { theme } from '../../common/theme';

const styles = StyleSheet.create({
  container: {
    backgroundColor: theme.colors.background.primary,
    paddingHorizontal: theme.spacing.medium
  },
  text: {
    color: theme.colors.font.primary,
    fontFamily: theme.fonts.medium,
    fontSize: theme.fontSizes.l
  },
  button: {
    backgroundColor: theme.colors.primary,
    borderRadius: theme.borderRadius.medium,
    paddingVertical: theme.spacing.small
  },
  error: {
    color: theme.colors.error,
    fontSize: theme.fontSizes.s
  },
  success: {
    color: theme.colors.success,
    fontSize: theme.fontSizes.m
  }
});
```

**Theme Color Palette:**
```typescript
// Available theme colors
theme.colors = {
  primary: '#007AFF',           // Primary action color
  secondary: '#5AC8FA',         // Secondary actions
  error: '#FF3B30',             // Error states
  success: '#34C759',           // Success states
  warning: '#FF9500',           // Warning states
  background: {
    primary: '#FFFFFF',
    secondary: '#F5F5F5',
    default: '#FFFFFF'
  },
  font: {
    primary: '#000000',
    secondary: '#666666',
    base: '#333333'
  },
  border: {
    primary: '#E0E0E0',
    secondary: '#F0F0F0'
  },
  shadow: '#000000'
};

theme.spacing = {
  xs: 4,
  s: 8,
  m: 16,
  l: 24,
  xl: 32,
  xxl: 48
};

theme.fontSizes = {
  xs: 10,
  s: 12,
  m: 14,
  l: 16,
  xl: 18,
  xxl: 20
};

theme.fonts = {
  light: 'Roboto-Light',
  regular: 'Roboto-Regular',
  medium: 'Roboto-Medium',
  bold: 'Roboto-Bold'
};

theme.borderRadius = {
  small: 4,
  medium: 8,
  large: 16
};
```

**Responsive Styling with moderateScale:**
```typescript
import { moderateScale } from 'react-native-size-matters';

const styles = StyleSheet.create({
  container: {
    paddingHorizontal: moderateScale(20),
    paddingVertical: moderateScale(15)
  },
  title: {
    fontSize: moderateScale(24),
    fontWeight: '600',
    marginBottom: moderateScale(12)
  },
  button: {
    height: moderateScale(50),
    paddingHorizontal: moderateScale(20),
    borderRadius: moderateScale(8)
  },
  card: {
    marginVertical: moderateScale(10),
    paddingHorizontal: moderateScale(16),
    borderRadius: moderateScale(12)
  }
});

// moderateScale scales values proportionally based on device screen size
// Ensures UI looks good on all device sizes
```

**Dynamic Theming (Light/Dark Mode):**
```typescript
import { useColorScheme } from 'react-native';

function ThemedComponent() {
  const colorScheme = useColorScheme();
  
  const styles = StyleSheet.create({
    container: {
      backgroundColor: colorScheme === 'dark' 
        ? theme.colors.dark.background 
        : theme.colors.light.background,
      color: colorScheme === 'dark'
        ? theme.colors.dark.text
        : theme.colors.light.text
    }
  });
  
  return (
    <View style={styles.container}>
      {/* Component content */}
    </View>
  );
}
```

## When Stuck

1. **State not updating?** Check:
   - Action dispatched with `dispatch()` not just returned
   - Reducer is whitelisted for persistence (if data should survive reload)
   - Saga is not silently catching errors
2. **Component not re-rendering?** Check:
   - Selector is not creating new references unnecessarily (use reselect if complex)
   - Redux middleware isn't paused (check connectivity state for installers/offline users)
3. **Async operations failing?** Check:
   - Loader keys are consistent between dispatch and selector
   - API errors are caught and failedLoadingAction is dispatched
   - Offline connectivity check passed before API call
4. **API calls timing out?** Check:
   - Request timeout is 20 seconds default (ABORT_CONTROLLER_TIMEOUT)
   - Network might be slow; check connectivity state
   - API endpoint might be down (check Sentry logs)
5. **401 errors not refreshing token?** Check:
   - Token refresh is automatic in response interceptor
   - Request is being retried after token refresh
   - If token can't refresh, user is logged out automatically

## CodePush & Over-The-Air (OTA) Updates

### Overview

CodePush is a cloud service that allows React Native developers to deliver Over-The-Air (OTA) updates directly to user devices — without going through App Store or Play Store re-approvals.

This system enables quick deployment of:
- 🔧 Bug Fixes
- 🎨 UI/UX Improvements
- 📜 Config / Translation Updates
- ⚡ Hotfix Patches

**Note**: CodePush can only update JavaScript and assets — native code or SDK updates still require a full release.

### How CodePush Works

When the app launches or resumes, it checks the configured CodePush server for available updates.

If an update is found:
1. The new JS bundle and assets are downloaded
2. Updates are installed based on configuration:
   - **Mandatory** → applied immediately
   - **Optional** → applied on next restart

### Integration in KeylessRN

We use the package `@code-push-next/react-native-code-push` for integration.

**OTA Acquisition Servers:**

| Environment | OTA URL |
|---|---|
| Production | https://ota.rently.com/api/codepush/acquisition |
| Byte | https://ota.rentlybyte.com/api/codepush/acquisition |

### Environment Configuration

All environment settings are centralized in [config.ts](config.ts), including:
- API base URLs
- CodePush channels and enable flags
- Sentry DSNs
- Flavor-specific identifiers
- Platform-specific deployment keys
- Armor client IDs

**Runtime Switching (No Rebuild Needed):**
```typescript
ENVIRONMENT = "production";
CODE_PUSH_CHANNEL = "Production";
IS_CODE_PUSH_ENABLED = true;
FLAVOUR = "bellanet";
```

This eliminates the need for `.env` files or AppCenter prebuilds.

### Key UI Components

**DeveloperOptions.tsx**
- Developers/QE can:
  - Change CodePush channel
  - Enable/disable CodePush
  - Switch environment via picker
- ⚠️ Disabled in production environment

**CodePushProgressBar.tsx**
- Listens to lifecycle events:
  - CHECKING_FOR_UPDATE
  - DOWNLOADING_PACKAGE
  - INSTALLING_UPDATE
  - UPDATE_INSTALLED
- Displays snackbar progress indicator (non-production only)

**VersionNumberWithDeveloperOptions**
- Shows native app version and CodePush version in parentheses
- Helps identify active JS bundle

### CodePush Channel Management

Each flavor + platform combination has its own deployment key in [config.ts](config.ts).

**Release Channels:**

| Channel | Purpose |
|---|---|
| Development | Local/dev testing |
| Evolution | Evolution QE testing |
| Avatar | Avatar QE testing |
| Matrix | Matrix QE testing |
| Staging | Pre-production testing |
| Production | Live user environment |

**⚠️ Important for Production Builds:**

| Flag | Value |
|---|---|
| CODE_PUSH_CHANNEL | Production |
| IS_CODE_PUSH_ENABLED | true |

**Note**: This may cause conflicts when using production builds for QE — check with the team if logic adjustments are required.

### Deployment Automation Scripts

**rnotaDeploy.sh** - Automates OTA deployments for all flavours
- Authenticates using deployment password (CodeRed)
- Loads configuration (rnotaDeploy.cfg)
- Builds JS bundle for each flavor
- Runs OTA deployment via `rnota codepush release`
- Uploads sourcemaps to Sentry
- Sends notifications via Google Chat

**rnotaKeys.sh** - Lists all CodePush deployment keys
- Use this to debug environment mismatches
- Verify configuration across all flavours and platforms

**TuyaConfig.sh** - Automates Tuya SDK configuration
- Detects SDK type based on flavor & environment
- Modifies Android (Gradle, Manifest, MainApplication.kt)
- Modifies iOS (imports, SDK refs)
- Manages SDK version (official vs unofficial)
- Updates ProGuard rules
- Replaces iOS SDK assets

**copy_flavour_icons.sh** - Manages flavor-specific icons
- Location: `src/common/theme/flavours/<flavour>/icons/`
- Usage: `bash copy_flavour_icons.sh <flavour>`
- Removes default icons and copies flavor-specific ones

### How to Release

**Regular Native Build (Android/iOS):**
```bash
yarn setup
yarn
yarn cleanAndroid     # or cleaniOS
yarn build-android    # or build via Xcode
```

No manual `.env` setup needed — handled dynamically by [config.ts](config.ts).

**OTA CodePush Release (Automated - Recommended):**

1. Prepare `rnotaDeploy.cfg`:
```bash
keyless=1
progress=0
coveyhomes=0
bellanet=0
version="17.5.0"
prepare_for_prod_release=0
mandatory_update=0
deployment_channel="Staging"
deploymentDescription="Staging Release"
release_ios=1
release_android=1
run_in_parallel=1
```

2. Run:
```bash
yarn ota
```

This will:
- Validate configuration
- Build & deploy OTA bundle
- Upload sourcemaps
- Send notifications

**Manual CodePush Release (Not Recommended):**
1. Ensure branch is clean (no uncommitted changes)
2. Verify Tuya version matches previous release
3. Prepare icons: `bash copy_flavour_icons.sh <flavour>`
4. Run deployment:
```bash
rnota codepush release react "$RNOTA_APP_NAME" "$platform" -t "$version" \
 -d "$deployment_channel" --description "$deploymentDescription" \
 $mandatory_update_flag $sourceMapOutputFlag
```
5. Upload sourcemaps & notify team

### Versioning & Best Practices

**Native App Versioning:**
- Increment version for each native build
- **Major (X.0.0)**: Breaking changes
- **Minor (1.X.0)**: New features
- **Patch (1.0.X)**: Bug fixes
- Each native build must have unique version
- Reset CodePush version to empty for new native builds

**CodePush Versioning:**
- Increment single-number version: 1, 2, 3…
- Represents updates for specific native build
- Every OTA release must have own version
- Base CodePush releases on latest native build

**Release Rules:**
- Use dedicated release branch per CodePush update
- Track changes in Confluence
- Release owner must verify native & CodePush versions before deployment
- Test all OTA updates in staging before production

**🚨 Critical Warning**
Improper versioning or reusing CodePush versions can overwrite user changes. **Always perform a CodePush release from the latest released branch** — whether it's a previous CodePush or native release. Failing to follow this can result in lost changes and unexpected issues.

### Troubleshooting Guide

| Issue | Possible Cause | Fix |
|---|---|---|
| App not receiving updates | Wrong CodePush key or channel; CodePush disabled | Verify CODE_PUSH_KEY in config.ts; enable CodePush |
| Update stuck at 0% | Network issue | Check asset URLs and retry |
| App crashes post-update | Breaking JS change | Test in staging first |
| App always installs CodePush and code is removed | CodePush release for same version as build | For Production: Increment version in package.json and native files. For Dev: Disable CodePush or use different channel |

---

**Last updated**: 2025-01-29  
**Maintainer**: Rently Development Team
