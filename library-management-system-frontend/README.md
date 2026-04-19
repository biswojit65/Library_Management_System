# Library Management System - Frontend

A modern React TypeScript frontend for the Library Management System.

## Features

- 🎨 **Modern UI/UX**: Built with Tailwind CSS and Lucide React icons
- 🌙 **Dark/Light Mode**: Toggle between dark and light themes
- 📱 **Responsive Design**: Mobile-first approach with responsive layouts
- 🔐 **Authentication**: JWT-based authentication with protected routes
- 📊 **Real-time Data**: React Query for efficient data fetching and caching
- 🎯 **Type Safety**: Full TypeScript support with strict mode
- 🚀 **Performance**: Code splitting, lazy loading, and optimized rendering
- ♿ **Accessibility**: WCAG compliant components and keyboard navigation

## Tech Stack

- **React 18** - Latest React with concurrent features
- **TypeScript** - Type-safe development
- **React Router v6** - Client-side routing
- **React Query** - Data fetching and caching
- **Tailwind CSS** - Utility-first CSS framework
- **Lucide React** - Beautiful icons
- **Formik + Yup** - Form handling and validation
- **React Hot Toast** - Toast notifications
- **Framer Motion** - Animations (optional)

## Getting Started

### Prerequisites

- Node.js 18+
- npm or yarn
- Backend API running (see backend README)

### Installation

1. Install dependencies:

```bash
npm install
```

2. Create environment file:

```bash
cp .env.example .env
```

3. Update environment variables in `.env`:

```env
VITE_API_URL=http://localhost:8080/api/v1

```

4. Start development server:

```bash
npm start
```

The application will be available at `http://localhost:3000`

## Available Scripts

- `npm run dev` - Start Vite development server
- `npm run build` - Build production bundle with Vite
- `npm run test` - Run tests with Vitest
- `npm run test:coverage` - Run tests with coverage via Vitest
- `npm run lint` - Run ESLint
- `npm run lint:fix` - Fix ESLint errors
- `npm run format` - Format code with Prettier
- `npm run type-check` - Run TypeScript type checking

## Project Structure

```
src/
├── components/          # Reusable UI components
│   ├── Auth/           # Authentication components
│   ├── Layout/         # Layout components
│   └── UI/             # Generic UI components
├── contexts/           # React contexts
├── hooks/              # Custom React hooks
├── pages/              # Page components
│   ├── Admin/          # Admin pages
│   ├── Auth/           # Authentication pages
│   ├── Books/          # Book-related pages
│   ├── Borrows/        # Borrow-related pages
│   ├── Dashboard/      # Dashboard pages
│   ├── Profile/        # User profile pages
│   └── Reservations/   # Reservation pages
├── services/           # API services
├── types/              # TypeScript type definitions
├── utils/              # Utility functions
├── App.tsx             # Main app component
└── index.tsx           # Entry point
```

## Key Features

### Authentication

- Login/Register forms with validation
- JWT token management
- Protected routes
- Role-based access control

### Dashboard

- Overview statistics
- Recent activity
- Quick actions
- Real-time updates

### Book Management

- Browse books with search and filters
- Book details with borrowing/reservation
- Grid and list view modes
- Pagination

### User Features

- Borrow books
- Return books
- Make reservations
- View borrowing history
- Profile management

### Admin Features

- Admin dashboard with statistics
- User management (placeholder)
- Book management (placeholder)
- Reports (placeholder)

## Development

### Code Style

- ESLint and Prettier configured
- TypeScript strict mode enabled
- Consistent naming conventions
- Component documentation

### State Management

- React Query for server state
- Context API for global state
- Local state with useState/useReducer

### Styling

- Tailwind CSS for styling
- CSS variables for theming
- Responsive design patterns
- Dark mode support

## Testing

```bash
# Run tests
npm test

# Run tests with coverage
npm run test:coverage

# Run tests in watch mode
npm test -- --watch
```

## Building for Production

```bash
# Build the application
npm run build

# Preview production build
npx serve -s build
```

## Deployment

The application can be deployed to any static hosting service:

- Vercel
- Netlify
- AWS S3
- GitHub Pages

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is licensed under the MIT License.
