# Frontend para Pais - DailyCare

## 📱 Visão Geral

A aplicação frontend para pais é uma SPA (Single Page Application) moderna desenvolvida em React, focada em proporcionar uma experiência intuitiva e responsiva para acompanhamento do desenvolvimento das crianças.

---

## 🏗️ Estrutura do Projeto

```
parent-app/
├── public/
│   ├── icons/              # Ícones PWA
│   ├── manifest.json       # PWA manifest
│   └── robots.txt
├── src/
│   ├── assets/             # Imagens, fontes, etc.
│   │   ├── images/
│   │   └── fonts/
│   ├── components/         # Componentes reutilizáveis
│   │   ├── common/         # Componentes genéricos
│   │   │   ├── Button/
│   │   │   ├── Card/
│   │   │   ├── Input/
│   │   │   ├── Modal/
│   │   │   └── Loader/
│   │   ├── layout/         # Componentes de layout
│   │   │   ├── Header/
│   │   │   ├── Sidebar/
│   │   │   ├── Footer/
│   │   │   └── Navigation/
│   │   └── features/       # Componentes específicos
│   │       ├── activities/
│   │       ├── health/
│   │       ├── milestones/
│   │       └── reports/
│   ├── features/           # Features organizadas por domínio
│   │   ├── auth/
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   ├── services/
│   │   │   └── types/
│   │   ├── dashboard/
│   │   ├── activities/
│   │   ├── children/
│   │   ├── health/
│   │   ├── milestones/
│   │   ├── reports/
│   │   └── notifications/
│   ├── hooks/              # Custom hooks globais
│   │   ├── useAuth.ts
│   │   ├── useTheme.ts
│   │   ├── useNotification.ts
│   │   └── useLocalStorage.ts
│   ├── layouts/            # Layouts de páginas
│   │   ├── AuthLayout.tsx
│   │   ├── MainLayout.tsx
│   │   └── EmptyLayout.tsx
│   ├── pages/              # Páginas da aplicação
│   │   ├── auth/
│   │   │   ├── Login.tsx
│   │   │   ├── Register.tsx
│   │   │   └── ForgotPassword.tsx
│   │   ├── dashboard/
│   │   │   └── Dashboard.tsx
│   │   ├── children/
│   │   │   ├── ChildrenList.tsx
│   │   │   └── ChildDetail.tsx
│   │   ├── activities/
│   │   │   ├── Activities.tsx
│   │   │   └── ActivityDetail.tsx
│   │   ├── health/
│   │   │   ├── HealthRecords.tsx
│   │   │   └── Vaccinations.tsx
│   │   ├── milestones/
│   │   │   └── Milestones.tsx
│   │   └── reports/
│   │       └── Reports.tsx
│   ├── routes/             # Configuração de rotas
│   │   ├── index.tsx
│   │   ├── PrivateRoute.tsx
│   │   └── PublicRoute.tsx
│   ├── services/           # Serviços e API clients
│   │   ├── api/
│   │   │   ├── client.ts
│   │   │   ├── auth.api.ts
│   │   │   ├── children.api.ts
│   │   │   ├── activities.api.ts
│   │   │   └── health.api.ts
│   │   └── signalr/
│   │       └── notifications.hub.ts
│   ├── store/              # Redux store
│   │   ├── index.ts
│   │   ├── hooks.ts
│   │   └── slices/
│   │       ├── authSlice.ts
│   │       ├── childrenSlice.ts
│   │       ├── activitiesSlice.ts
│   │       └── uiSlice.ts
│   ├── styles/             # Estilos globais
│   │   ├── global.css
│   │   ├── theme.ts        # Tema MUI
│   │   └── tailwind.css
│   ├── types/              # TypeScript types globais
│   │   ├── api.types.ts
│   │   ├── models.ts
│   │   └── enums.ts
│   ├── utils/              # Utilitários
│   │   ├── date.utils.ts
│   │   ├── format.utils.ts
│   │   ├── validation.utils.ts
│   │   └── storage.utils.ts
│   ├── App.tsx             # Componente raiz
│   ├── main.tsx            # Entry point
│   └── vite-env.d.ts       # Vite types
├── .env.example            # Exemplo de variáveis de ambiente
├── .eslintrc.cjs          # ESLint config
├── .prettierrc            # Prettier config
├── index.html             # HTML entry
├── package.json
├── tsconfig.json          # TypeScript config
├── tsconfig.node.json
├── tailwind.config.js     # Tailwind config
├── vite.config.ts         # Vite config
└── README.md
```

---

## 🎨 Sistema de Design

### Tema (Theme)

#### Configuração de Cores

```typescript
// src/styles/theme.ts
import { createTheme, ThemeOptions } from '@mui/material/styles';

const lightThemeOptions: ThemeOptions = {
  palette: {
    mode: 'light',
    primary: {
      main: '#4C63D2',
      light: '#6B7FE6',
      dark: '#3B4FA8',
      contrastText: '#FFFFFF',
    },
    secondary: {
      main: '#FF5252',
      light: '#FF8A80',
      dark: '#C50E29',
      contrastText: '#FFFFFF',
    },
    background: {
      default: '#F5F7FA',
      paper: '#FFFFFF',
    },
    text: {
      primary: '#2C3E50',
      secondary: '#7F8C8D',
    },
    success: {
      main: '#4CAF50',
    },
    warning: {
      main: '#FF9800',
    },
    error: {
      main: '#F44336',
    },
    info: {
      main: '#2196F3',
    },
  },
  typography: {
    fontFamily: '"Inter", "Roboto", "Helvetica", "Arial", sans-serif',
    h1: {
      fontSize: '2.5rem',
      fontWeight: 600,
    },
    h2: {
      fontSize: '2rem',
      fontWeight: 600,
    },
    h3: {
      fontSize: '1.75rem',
      fontWeight: 600,
    },
    h4: {
      fontSize: '1.5rem',
      fontWeight: 600,
    },
    h5: {
      fontSize: '1.25rem',
      fontWeight: 600,
    },
    h6: {
      fontSize: '1rem',
      fontWeight: 600,
    },
  },
  shape: {
    borderRadius: 8,
  },
  components: {
    MuiButton: {
      styleOverrides: {
        root: {
          textTransform: 'none',
          fontWeight: 500,
        },
      },
    },
    MuiCard: {
      styleOverrides: {
        root: {
          boxShadow: '0 2px 8px rgba(0,0,0,0.1)',
        },
      },
    },
  },
};

const darkThemeOptions: ThemeOptions = {
  palette: {
    mode: 'dark',
    primary: {
      main: '#6B7FE6',
      light: '#8A9BF0',
      dark: '#4C63D2',
      contrastText: '#FFFFFF',
    },
    secondary: {
      main: '#FF8A80',
      light: '#FFB3AD',
      dark: '#FF5252',
      contrastText: '#FFFFFF',
    },
    background: {
      default: '#121212',
      paper: '#1E1E1E',
    },
    text: {
      primary: '#FFFFFF',
      secondary: '#B0B0B0',
    },
  },
};

export const lightTheme = createTheme(lightThemeOptions);
export const darkTheme = createTheme(darkThemeOptions);
```

#### Deteção de Tema do Sistema

```typescript
// src/hooks/useTheme.ts
import { useState, useEffect } from 'react';
import { useLocalStorage } from './useLocalStorage';

type ThemeMode = 'light' | 'dark' | 'system';

export const useTheme = () => {
  const [themePreference, setThemePreference] = useLocalStorage<ThemeMode>(
    'theme-preference',
    'system'
  );
  const [actualTheme, setActualTheme] = useState<'light' | 'dark'>('light');

  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    
    const updateTheme = () => {
      if (themePreference === 'system') {
        setActualTheme(mediaQuery.matches ? 'dark' : 'light');
      } else {
        setActualTheme(themePreference);
      }
    };

    updateTheme();
    mediaQuery.addEventListener('change', updateTheme);
    
    return () => mediaQuery.removeEventListener('change', updateTheme);
  }, [themePreference]);

  return {
    themePreference,
    actualTheme,
    setThemePreference,
  };
};
```

### Componentes Base

#### Button Component

```typescript
// src/components/common/Button/Button.tsx
import { Button as MuiButton, ButtonProps as MuiButtonProps } from '@mui/material';
import { FC } from 'react';

export interface ButtonProps extends MuiButtonProps {
  loading?: boolean;
}

export const Button: FC<ButtonProps> = ({ loading, children, disabled, ...props }) => {
  return (
    <MuiButton
      {...props}
      disabled={disabled || loading}
      className={`${props.className} ${loading ? 'opacity-70' : ''}`}
    >
      {loading ? 'A carregar...' : children}
    </MuiButton>
  );
};
```

---

## 🔐 Autenticação

### Auth Service

```typescript
// src/services/api/auth.api.ts
import { apiClient } from './client';

export interface LoginRequest {
  email: string;
  password: string;
}

export interface LoginResponse {
  accessToken: string;
  refreshToken: string;
  user: {
    id: string;
    name: string;
    email: string;
    role: string;
  };
}

export interface RegisterRequest {
  name: string;
  email: string;
  password: string;
  confirmPassword: string;
}

export const authApi = {
  login: async (credentials: LoginRequest): Promise<LoginResponse> => {
    const response = await apiClient.post<LoginResponse>('/auth/login', credentials);
    return response.data;
  },

  register: async (data: RegisterRequest): Promise<void> => {
    await apiClient.post('/auth/register', data);
  },

  refreshToken: async (refreshToken: string): Promise<LoginResponse> => {
    const response = await apiClient.post<LoginResponse>('/auth/refresh', { refreshToken });
    return response.data;
  },

  logout: async (): Promise<void> => {
    await apiClient.post('/auth/logout');
  },

  forgotPassword: async (email: string): Promise<void> => {
    await apiClient.post('/auth/forgot-password', { email });
  },

  resetPassword: async (token: string, newPassword: string): Promise<void> => {
    await apiClient.post('/auth/reset-password', { token, newPassword });
  },
};
```

### Auth Slice (Redux)

```typescript
// src/store/slices/authSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import { authApi, LoginRequest, LoginResponse } from '@/services/api/auth.api';

interface AuthState {
  user: LoginResponse['user'] | null;
  accessToken: string | null;
  refreshToken: string | null;
  isAuthenticated: boolean;
  loading: boolean;
  error: string | null;
}

const initialState: AuthState = {
  user: null,
  accessToken: localStorage.getItem('accessToken'),
  refreshToken: localStorage.getItem('refreshToken'),
  isAuthenticated: !!localStorage.getItem('accessToken'),
  loading: false,
  error: null,
};

export const login = createAsyncThunk(
  'auth/login',
  async (credentials: LoginRequest, { rejectWithValue }) => {
    try {
      const response = await authApi.login(credentials);
      localStorage.setItem('accessToken', response.accessToken);
      localStorage.setItem('refreshToken', response.refreshToken);
      return response;
    } catch (error: any) {
      return rejectWithValue(error.response?.data?.message || 'Erro ao fazer login');
    }
  }
);

export const logout = createAsyncThunk('auth/logout', async () => {
  await authApi.logout();
  localStorage.removeItem('accessToken');
  localStorage.removeItem('refreshToken');
});

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    setCredentials: (state, action: PayloadAction<LoginResponse>) => {
      state.user = action.payload.user;
      state.accessToken = action.payload.accessToken;
      state.refreshToken = action.payload.refreshToken;
      state.isAuthenticated = true;
    },
    clearAuth: (state) => {
      state.user = null;
      state.accessToken = null;
      state.refreshToken = null;
      state.isAuthenticated = false;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(login.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(login.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload.user;
        state.accessToken = action.payload.accessToken;
        state.refreshToken = action.payload.refreshToken;
        state.isAuthenticated = true;
      })
      .addCase(login.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload as string;
      })
      .addCase(logout.fulfilled, (state) => {
        state.user = null;
        state.accessToken = null;
        state.refreshToken = null;
        state.isAuthenticated = false;
      });
  },
});

export const { setCredentials, clearAuth } = authSlice.actions;
export default authSlice.reducer;
```

---

## 📄 Páginas Principais

### Dashboard

```typescript
// src/pages/dashboard/Dashboard.tsx
import { FC } from 'react';
import { Grid, Card, CardContent, Typography, Box } from '@mui/material';
import { useNavigate } from 'react-router-dom';
import { ChildCard } from '@/features/children/components/ChildCard';
import { ActivitySummary } from '@/features/activities/components/ActivitySummary';
import { UpcomingEvents } from '@/features/health/components/UpcomingEvents';

export const Dashboard: FC = () => {
  const navigate = useNavigate();

  return (
    <Box className="p-6">
      <Typography variant="h4" className="mb-6 font-bold">
        Painel Principal
      </Typography>

      <Grid container spacing={3}>
        {/* Crianças */}
        <Grid item xs={12} md={8}>
          <Card>
            <CardContent>
              <Typography variant="h6" className="mb-4">
                As Minhas Crianças
              </Typography>
              <ChildCard />
            </CardContent>
          </Card>
        </Grid>

        {/* Eventos Próximos */}
        <Grid item xs={12} md={4}>
          <Card>
            <CardContent>
              <Typography variant="h6" className="mb-4">
                Próximos Eventos
              </Typography>
              <UpcomingEvents />
            </CardContent>
          </Card>
        </Grid>

        {/* Resumo de Atividades */}
        <Grid item xs={12}>
          <Card>
            <CardContent>
              <Typography variant="h6" className="mb-4">
                Atividades Recentes
              </Typography>
              <ActivitySummary />
            </CardContent>
          </Card>
        </Grid>
      </Grid>
    </Box>
  );
};
```

### Lista de Atividades

```typescript
// src/pages/activities/Activities.tsx
import { FC, useState } from 'react';
import {
  Box,
  Typography,
  Tabs,
  Tab,
  Card,
  CardContent,
} from '@mui/material';
import { FeedingActivities } from '@/features/activities/components/FeedingActivities';
import { SleepActivities } from '@/features/activities/components/SleepActivities';
import { DiaperActivities } from '@/features/activities/components/DiaperActivities';

export const Activities: FC = () => {
  const [activeTab, setActiveTab] = useState(0);

  return (
    <Box className="p-6">
      <Typography variant="h4" className="mb-6 font-bold">
        Atividades
      </Typography>

      <Card>
        <Tabs
          value={activeTab}
          onChange={(_, newValue) => setActiveTab(newValue)}
          className="border-b"
        >
          <Tab label="Alimentação" />
          <Tab label="Sono" />
          <Tab label="Fraldas" />
          <Tab label="Exercícios" />
        </Tabs>

        <CardContent>
          {activeTab === 0 && <FeedingActivities />}
          {activeTab === 1 && <SleepActivities />}
          {activeTab === 2 && <DiaperActivities />}
          {activeTab === 3 && <div>Exercícios</div>}
        </CardContent>
      </Card>
    </Box>
  );
};
```

---

## 🔄 Gestão de Estado

### Redux Store Setup

```typescript
// src/store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import authReducer from './slices/authSlice';
import childrenReducer from './slices/childrenSlice';
import activitiesReducer from './slices/activitiesSlice';
import uiReducer from './slices/uiSlice';

export const store = configureStore({
  reducer: {
    auth: authReducer,
    children: childrenReducer,
    activities: activitiesReducer,
    ui: uiReducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: false,
    }),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### Typed Hooks

```typescript
// src/store/hooks.ts
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './index';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

---

## 🌐 Rotas

### Configuração de Rotas

```typescript
// src/routes/index.tsx
import { createBrowserRouter } from 'react-router-dom';
import { MainLayout } from '@/layouts/MainLayout';
import { AuthLayout } from '@/layouts/AuthLayout';
import { PrivateRoute } from './PrivateRoute';
import { Dashboard } from '@/pages/dashboard/Dashboard';
import { Login } from '@/pages/auth/Login';
import { Activities } from '@/pages/activities/Activities';
// ... outras importações

export const router = createBrowserRouter([
  {
    path: '/auth',
    element: <AuthLayout />,
    children: [
      { path: 'login', element: <Login /> },
      { path: 'register', element: <Register /> },
      { path: 'forgot-password', element: <ForgotPassword /> },
    ],
  },
  {
    path: '/',
    element: <PrivateRoute><MainLayout /></PrivateRoute>,
    children: [
      { index: true, element: <Dashboard /> },
      { path: 'children', element: <ChildrenList /> },
      { path: 'children/:id', element: <ChildDetail /> },
      { path: 'activities', element: <Activities /> },
      { path: 'health', element: <HealthRecords /> },
      { path: 'milestones', element: <Milestones /> },
      { path: 'reports', element: <Reports /> },
      { path: 'settings', element: <Settings /> },
    ],
  },
]);
```

---

## 📱 PWA (Progressive Web App)

### Manifest

```json
// public/manifest.json
{
  "name": "DailyCare - Gestão Infantil",
  "short_name": "DailyCare",
  "description": "Aplicação para gestão do dia a dia de crianças",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#FFFFFF",
  "theme_color": "#4C63D2",
  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

---

## 🔔 Notificações em Tempo Real

### SignalR Hub Connection

```typescript
// src/services/signalr/notifications.hub.ts
import * as signalR from '@microsoft/signalr';

class NotificationHub {
  private connection: signalR.HubConnection | null = null;

  connect(accessToken: string) {
    this.connection = new signalR.HubConnectionBuilder()
      .withUrl(`${import.meta.env.VITE_API_URL}/hubs/notifications`, {
        accessTokenFactory: () => accessToken,
      })
      .withAutomaticReconnect()
      .build();

    this.connection.start().catch((err) => console.error('SignalR Error:', err));

    this.connection.on('ReceiveNotification', (notification) => {
      // Despachar para Redux ou mostrar notificação
      console.log('Nova notificação:', notification);
    });
  }

  disconnect() {
    this.connection?.stop();
  }
}

export const notificationHub = new NotificationHub();
```

---

## 🧪 Testes

### Exemplo de Teste de Componente

```typescript
// src/components/common/Button/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button Component', () => {
  it('deve renderizar com texto correto', () => {
    render(<Button>Clique aqui</Button>);
    expect(screen.getByText('Clique aqui')).toBeInTheDocument();
  });

  it('deve chamar onClick quando clicado', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Clique</Button>);
    
    fireEvent.click(screen.getByText('Clique'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('deve mostrar loading state', () => {
    render(<Button loading>Clique</Button>);
    expect(screen.getByText('A carregar...')).toBeInTheDocument();
  });
});
```

---

## 📦 Build e Deploy

### Variáveis de Ambiente

```bash
# .env.example
VITE_API_URL=http://localhost:5000/api
VITE_SIGNALR_URL=http://localhost:5000
VITE_ENVIRONMENT=development
```

### Build para Produção

```bash
# Build
npm run build

# Preview
npm run preview
```

---

**Próximo documento**: [Frontend Administrativo](./05-FRONTEND-ADMIN.md)  
**Anterior**: [Stack Tecnológica](./03-STACK-TECNOLOGICA.md)  
**Voltar ao**: [Índice](./00-INDICE.md)
