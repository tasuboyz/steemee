# Cur8.fun Architecture Document

## 1. Overview

Cur8.fun (previously SteemGram) is a modern, client-side web application built to interact with the Steem blockchain. It provides a user-friendly interface for browsing, creating, and interacting with content on the Steem ecosystem. The application is built entirely with vanilla JavaScript (ES6+), HTML, and CSS, requiring no build tools or server-side components for basic operation.

## 2. Architectural Pattern

The application follows a component-based architecture with clear separation of concerns:

- **MVC Pattern**: Model-View-Controller pattern is used throughout the application
  - **Models**: Represent data structures
  - **Views**: Handle UI rendering and user interaction
  - **Controllers**: Manage business logic and data flow
  - **Services**: Provide API interactions and blockchain operations

## 3. Core Components

### 3.1 Routing System

The routing system (`utils/Router.js`) is a client-side router that enables SPA (Single Page Application) navigation:

- **URL Handling**: Supports both hash-based (`#/path`) and history API routing
- **Route Definitions**: Each route is mapped to a specific view
- **Parameter Extraction**: Extracts URL parameters (e.g., `/tag/:tag`) for dynamic routing
- **Navigation Guard**: Guards routes that require authentication
- **Route History**: Tracks navigation history for back/forward functionality

```javascript
// Example route registration
router
  .addRoute('/', HomeView, { tag: 'trending' })
  .addRoute('/login', LoginView)
  .addRoute('/tag/:tag', TagView)
  .addRoute('/@:username', ProfileView)
  .setNotFound(NotFoundView);
```

### 3.2 View System

Views (`views/`) are responsible for rendering pages and managing user interactions:

- **Base View**: All views inherit from `View.js` base class
- **Lifecycle Methods**: `render()` for initialization, `unmount()` for cleanup
- **Event Management**: Automatic event subscription tracking for cleanup
- **Component Composition**: Views often compose multiple smaller components

```javascript
class ProfileView extends View {
  constructor(params) {
    super(params);
    this.username = params.username;
  }
  
  async render(element) {
    // Render UI elements
    // Load data from services
    // Register event handlers
  }
  
  unmount() {
    // Clean up resources
    super.unmount();
  }
}
```

### 3.3 Component System

Components (`components/`) are reusable UI elements with their own lifecycle:

- **Base Component**: Components inherit from `Component.js` base class
- **Self-Contained**: Each component manages its own state and rendering
- **Event Management**: Components track event handlers for automatic cleanup
- **Composability**: Components can be nested to create complex UI structures

```javascript
class PostCard extends Component {
  constructor(parentElement, post, options) {
    super(parentElement, options);
    this.post = post;
  }
  
  render() {
    const element = document.createElement('div');
    element.className = 'post-card';
    // Build UI elements
    this.parentElement.appendChild(element);
    this.element = element;
    
    // Register event handlers
    this.registerEventHandler(element, 'click', this.handleClick.bind(this));
    
    return element;
  }
}
```

### 3.4 Services

Services (`services/`) handle API calls and blockchain interactions:

- **API Abstraction**: Hide complexities of blockchain interactions
- **Caching**: Implement caching for improved performance
- **Authentication**: Manage user credentials and sessions
- **Data Processing**: Transform raw blockchain data into app-friendly formats

Key services include:
- **SteemService**: Core interface to the Steem blockchain API
- **AuthService**: Handle user authentication and session management
- **ProfileService**: Manage user profile data
- **PostService**: Create, read, and manipulate posts
- **CommentService**: Handle comment operations
- **VoteService**: Manage voting operations
- **NotificationsService**: Track and display user notifications

### 3.5 Event System

The event system (`utils/EventEmitter.js`) enables loose coupling between components:

- **Event Subscription**: Components can subscribe to global events
- **Event Broadcasting**: Services and controllers can broadcast events
- **Automatic Cleanup**: Views track subscriptions for cleanup on unmount

```javascript
// Broadcasting an event
eventEmitter.emit('notification', {
  type: 'success',
  message: 'Post created successfully'
});

// Subscribing to an event
this.subscribe('auth:changed', this.updateUserStatus.bind(this));
```

## 4. Data Flow

### 4.1 Post Loading Flow

1. User navigates to a route (e.g., `/trending`)
2. Router initializes the appropriate view (e.g., `HomeView`)
3. View calls service methods (e.g., `steemService.getTrendingPosts()`)
4. Service fetches data from the blockchain
5. View receives data and renders UI elements
6. Components handle user interactions (votes, comments, etc.)

### 4.2 Authentication Flow

1. User navigates to `/login`
2. LoginView presents authentication options
3. User provides credentials
4. AuthService validates credentials with the blockchain
5. On success, AuthService stores session data
6. EventEmitter broadcasts 'auth:changed' event
7. UI components update to reflect authenticated state

## 5. Application Structure

```
/
├── assets/              # Static assets
│   ├── css/             # CSS stylesheets and modules
│   └── img/             # Images and graphics
├── components/          # Reusable UI components
│   ├── comments/        # Comment-related components
│   ├── post/            # Post display components
│   ├── profile/         # User profile components
│   └── wallet/          # Wallet and finance components
├── controllers/         # Business logic controllers
├── models/              # Data models
├── services/            # API and blockchain services
├── utils/               # Utility functions
└── views/               # Page views and layouts
```

## 6. UI Component Hierarchy

- `App` (Root container)
  - `NavigationBar`
    - `SearchBar`
    - `UserMenu`
  - `MainContent` (Router container)
    - Current View (e.g., `HomeView`, `ProfileView`, etc.)
      - View-specific components (e.g., `PostsList`, `ProfileHeader`)
        - Shared components (e.g., `PostCard`, `CommentThread`)

## 7. Key Features Implementation

### 7.1 Content Feeds

Multiple content views are implemented through specialized views:
- `HomeView`: Shows posts from different categories (trending, hot, new)
- `TagView`: Shows posts for a specific tag
- `CommunityView`: Shows posts within a Steem community
- `SearchView`: Shows search results for posts, users, or tags

### 7.2 Draft Management

The draft system allows users to save and manage post drafts:

- **Multi-Draft Support**: Users can have multiple drafts (up to 10)
- **Auto-save**: Posts are automatically saved as drafts
- **Draft Organization**: Users can view, edit, and delete drafts from the DraftsView

### 7.3 Wallet & Transactions

The wallet system provides financial information and transaction capabilities:

- **Balance Display**: Shows STEEM, SBD, and Steem Power balances
- **Resource Credits**: Monitors blockchain resource usage
- **Transaction History**: Shows history of transactions
- **Reward Claims**: Allows claiming of pending rewards

## 8. Security Considerations

- **XSS Protection**: Content sanitization using DOMPurify
- **Credential Management**: Secure handling of user credentials
- **Permission Handling**: Proper permission checks before transactions

## 9. Performance Optimizations

- **Infinite Scrolling**: Load content progressively as user scrolls
- **Lazy Loading**: Load images and content only when needed
- **Content Caching**: Cache frequently accessed blockchain data
- **Grid Controller**: Optimize layout calculations for content rendering

## 10. PWA Features

The application includes Progressive Web App features:

- **Service Worker**: Enables offline functionality
- **Cache API**: Caches app shell for faster loading
- **Update Notification**: Notifies users about app updates

## 11. Conclusion

Cur8.fun's architecture is designed for modularity, maintainability, and user experience. The component-based approach with clear separation of concerns allows for easy extension and maintenance. The application leverages vanilla JavaScript without dependencies on heavy frameworks, resulting in a lightweight yet powerful user interface for the Steem blockchain.
