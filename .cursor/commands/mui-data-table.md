# MUI Data Table

## Description
Guide for creating data tables using Material-UI (MUI) DataGrid component with sorting, filtering, and pagination.

## Style Rules
- Use MUI DataGrid or DataGridPro for complex tables
- Define column definitions with proper types
- Implement proper TypeScript interfaces for row data
- Use GridColDef for column configuration
- Handle loading and error states
- Implement pagination for large datasets
- Use proper theme integration
- Add accessible labels and ARIA attributes

## Example

```typescript
import React from 'react';
import { DataGrid, GridColDef, GridRenderCellParams } from '@mui/x-data-grid';
import { Box, Chip, IconButton } from '@mui/material';
import EditIcon from '@mui/icons-material/Edit';
import DeleteIcon from '@mui/icons-material/Delete';

interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
  status: 'active' | 'inactive';
  createdAt: string;
}

interface UserTableProps {
  users: User[];
  loading?: boolean;
  onEdit: (id: number) => void;
  onDelete: (id: number) => void;
}

export const UserTable: React.FC<UserTableProps> = ({
  users,
  loading = false,
  onEdit,
  onDelete,
}) => {
  const columns: GridColDef[] = [
    {
      field: 'id',
      headerName: 'ID',
      width: 70,
      sortable: true,
    },
    {
      field: 'name',
      headerName: 'Name',
      width: 200,
      sortable: true,
    },
    {
      field: 'email',
      headerName: 'Email',
      width: 250,
      sortable: true,
    },
    {
      field: 'role',
      headerName: 'Role',
      width: 120,
      renderCell: (params: GridRenderCellParams<User>) => {
        const colorMap = {
          admin: 'error',
          user: 'primary',
          guest: 'default',
        } as const;
        return (
          <Chip
            label={params.value}
            color={colorMap[params.value as keyof typeof colorMap]}
            size="small"
          />
        );
      },
    },
    {
      field: 'status',
      headerName: 'Status',
      width: 120,
      renderCell: (params: GridRenderCellParams<User>) => (
        <Chip
          label={params.value}
          color={params.value === 'active' ? 'success' : 'default'}
          size="small"
        />
      ),
    },
    {
      field: 'createdAt',
      headerName: 'Created',
      width: 150,
      valueFormatter: (params) => {
        return new Date(params.value).toLocaleDateString();
      },
    },
    {
      field: 'actions',
      headerName: 'Actions',
      width: 120,
      sortable: false,
      renderCell: (params: GridRenderCellParams<User>) => (
        <Box>
          <IconButton
            size="small"
            onClick={() => onEdit(params.row.id)}
            aria-label="edit"
          >
            <EditIcon fontSize="small" />
          </IconButton>
          <IconButton
            size="small"
            onClick={() => onDelete(params.row.id)}
            aria-label="delete"
          >
            <DeleteIcon fontSize="small" />
          </IconButton>
        </Box>
      ),
    },
  ];

  return (
    <Box sx={{ height: 600, width: '100%' }}>
      <DataGrid
        rows={users}
        columns={columns}
        loading={loading}
        pagination
        pageSizeOptions={[5, 10, 25, 50]}
        initialState={{
          pagination: { paginationModel: { pageSize: 10 } },
        }}
        checkboxSelection
        disableRowSelectionOnClick
      />
    </Box>
  );
};
```

## Usage

```typescript
import { UserTable } from './components/UserTable';

function App() {
  const users: User[] = [
    {
      id: 1,
      name: 'John Doe',
      email: 'john@example.com',
      role: 'admin',
      status: 'active',
      createdAt: '2024-01-15',
    },
    // ... more users
  ];

  const handleEdit = (id: number) => {
    console.log('Edit user:', id);
  };

  const handleDelete = (id: number) => {
    console.log('Delete user:', id);
  };

  return (
    <UserTable
      users={users}
      onEdit={handleEdit}
      onDelete={handleDelete}
    />
  );
}
```
