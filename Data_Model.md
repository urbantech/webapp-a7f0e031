# Data Model

# ZeroDB Data Model for WebApp Task Management System

```json
{
  "database_type": "zerodb",
  "version": "1.0.0",
  "application": "WebApp - Task Management System",
  "description": "A comprehensive task management application with user authentication, real-time updates, and admin panel capabilities",
  "multi_tenant": {
    "enabled": true,
    "strategy": "tenant_column",
    "reasoning": "User-based isolation for personal task management with potential for team/organization expansion"
  },
  "entities": [
    {
      "name": "User",
      "table_type": "sql",
      "description": "User accounts with authentication and profile information",
      "fields": [
        {
          "name": "id",
          "type": "UUID",
          "primary_key": true,
          "default": "gen_random_uuid()"
        },
        {
          "name": "email",
          "type": "String",
          "nullable": false,
          "unique": true,
          "indexed": true,
          "max_length": 255,
          "pattern": "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
        },
        {
          "name": "hashed_password",
          "type": "String",
          "nullable": false,
          "max_length": 255
        },
        {
          "name": "full_name",
          "type": "String",
          "nullable": false,
          "max_length": 255
        },
        {
          "name": "avatar_url",
          "type": "String",
          "nullable": true,
          "max_length": 500
        },
        {
          "name": "role",
          "type": "Enum",
          "values": ["admin", "user"],
          "default": "user",
          "nullable": false
        },
        {
          "name": "is_active",
          "type": "Boolean",
          "default": true,
          "nullable": false
        },
        {
          "name": "email_verified",
          "type": "Boolean",
          "default": false,
          "nullable": false
        },
        {
          "name": "last_login_at",
          "type": "DateTime",
          "nullable": true
        },
        {
          "name": "preferences",
          "type": "JSON",
          "default": "{}",
          "description": "User preferences like theme, notifications, etc."
        },
        {
          "name": "created_at",
          "type": "DateTime",
          "default": "now()",
          "nullable": false
        },
        {
          "name": "updated_at",
          "type": "DateTime",
          "default": "now()",
          "on_update": "now()",
          "nullable": false
        },
        {
          "name": "deleted_at",
          "type": "DateTime",
          "nullable": true,
          "description": "Soft delete timestamp"
        }
      ],
      "indexes": [
        {
          "name": "idx_user_email",
          "fields": ["email"],
          "type": "btree",
          "unique": true
        },
        {
          "name": "idx_user_active",
          "fields": ["is_active", "created_at"],
          "type": "btree",
          "where": "deleted_at IS NULL"
        },
        {
          "name": "idx_user_role",
          "fields": ["role"],
          "type": "btree",
          "where": "is_active = true"
        }
      ]
    },
    {
      "name": "UserSession",
      "table_type": "memory",
      "description": "Active user sessions with automatic expiration for real-time authentication",
      "ttl": 3600,
      "fields": [
        {
          "name": "session_id",
          "type": "String",
          "primary_key": true
        },
        {
          "name": "user_id",
          "type": "UUID",
          "nullable": false
        },
        {
          "name": "ip_address",
          "type": "INET",
          "nullable": true
        },
        {
          "name": "user_agent",
          "type": "String",
          "nullable": true
        },
        {
          "name": "last_activity",
          "type": "DateTime",
          "default": "now()"
        },
        {
          "name": "data",
          "type": "JSON",
          "default": "{}"
        }
      ]
    },
    {
      "name": "Task",
      "table_type": "sql",
      "description": "Core task entity with semantic search capability for intelligent task discovery",
      "multi_tenant": {
        "enabled": true,
        "strategy": "tenant_column",
        "tenant_field": "user_id"
      },
      "fields": [
        {
          "name": "id",
          "type": "UUID",
          "primary_key": true,
          "default": "gen_random_uuid()"
        },
        {
          "name": "user_id",
          "type": "UUID",
          "nullable": false,
          "indexed": true,
          "description": "Task owner (tenant isolation)"
        },
        {
          "name": "title",
          "type": "String",
          "nullable": false,
          "max_length": 255
        },
        {
          "name": "description",
          "type": "Text",
          "nullable": true,
          "description": "Detailed task description for semantic search"
        },
        {
          "name": "status",
          "type": "Enum",
          "values": ["todo", "in_progress", "completed", "archived"],
          "default": "todo",
          "nullable": false,
          "indexed": true
        },
        {
          "name": "priority",
          "type": "Enum",
          "values": ["low", "medium", "high", "urgent"],
          "default": "medium",
          "nullable": false,
          "indexed": true
        },
        {
          "name": "due_date",
          "type": "DateTime",
          "nullable": true,
          "indexed": true
        },
        {
          "name": "completed_at",
          "type": "DateTime",
          "nullable": true
        },
        {
          "name": "tags",
          "type": "Array",
          "array_type": "String",
          "default": "[]",
          "description": "Task categorization tags"
        },
        {
          "name": "category_id",
          "type": "UUID",
          "nullable": true,
          "indexed": true
        },
        {
          "name": "assigned_to",
          "type": "UUID",
          "nullable": true,
          "indexed": true,
          "description": "For future team collaboration features"
        },
        {
          "name": "parent_task_id",
          "type": "UUID",
          "nullable": true,
          "indexed": true,
          "description": "For subtasks hierarchy"
        },
        {
          "name": "position",
          "type": "Integer",
          "default": 0,
          "description": "For drag-and-drop ordering"
        },
        {
          "name": "metadata",
          "type": "JSON",
          "default": "{}",
          "description": "Additional custom fields"
        },
        {
          "name": "created_at",
          "type": "DateTime",
          "default": "now()",
          "nullable": false
        },
        {
          "name": "updated_at",
          "type": "DateTime",
          "default": "now()",
          "on_update": "now()",
          "nullable": false
        },
        {
          "name": "deleted_at",
          "type": "DateTime",
          "nullable": true
        }
      ],
      "vector_index": {
        "enabled": true,
        "field": "description",
        "dimensions": 1536,
        "distance_metric": "cosine",
        "purpose": "semantic_task_search",
        "metadata_fields": ["status", "priority", "tags", "category_id"],
        "enable_hybrid_search": true,
        "index_type": "hnsw",
        "index_params": {
          "m": 16,
          "ef_construction": 200
        },
        "description": "Enable intelligent task search like 'find tasks about project planning'"
      },
      "indexes": [
        {
          "name": "idx_task_user_status",
          "fields": ["user_id", "status", "created_at"],
          "type": "btree"
        },
        {
          "name": "idx_task_user_priority",
          "fields": ["user_id", "priority", "due_date"],
          "type": "btree"
        },
        {
          "name": "idx_task_due_date",
          "fields": ["user_id", "due_date"],
          "type": "btree",
          "where": "status != 'completed' AND status != 'archived' AND deleted_at IS NULL"
        },
        {
          "name": "idx_task_tags",
          "fields": ["tags"],
          "type": "gin"
        },
        {
          "name": "idx_task_category",
          "fields": ["category_id", "created_at"],
          "type": "btree"
        },
        {
          "name": "idx_task_parent",
          "fields": ["parent_task_id"],
          "type": "btree"
        },
        {
          "name": "idx_task_assigned",
          "fields": ["assigned_to", "status"],
          "type": "btree"
        }
      ],
      "relationships": [
        {
          "type": "many_to_one",
          "target": "User",
          "foreign_key": "user_id",
          "on_delete": "CASCADE"
        },
        {
          "type": "many_to_one",
          "target": "Category",
          "foreign_key": "category_id",
          "on_delete": "SET NULL"
        },
        {
          "type": "many_to_one",
          "target": "User",
          "foreign_key": "assigned_to",
          "on_delete": "SET NULL"
        },
        {
          "type": "many_to_one",
          "target": "Task",
          "foreign_key": "parent_task_id",
          "on_delete": "CASCADE",
          "description": "Self-referencing for subtasks"
        }
      ]
    },
    {
      "name": "Category",
      "table_type": "sql",
      "description": "Task categories for organization",
      "multi_tenant": {
        "enabled": true,
        "strategy": "tenant_column",
        "tenant_field": "user_id"
      },
      "fields": [
        {
          "name": "id",
          "type": "UUID",
          "primary_key": true,
          "default": "gen_random_uuid()"
        },
        {
          "name": "user_id",
          "type": "UUID",
          "nullable": false,
          "indexed": true
        },
        {
          "name": "name",
          "type": "String",
          "nullable": false,
          "max_length": 100
        },
        {
          "name": "color",
          "type": "String",
          "nullable": true,
          "max_length": 7,
          "description": "Hex color code for UI"
        },
        {
          "name": "icon",
          "type": "String",
          "nullable": true,
          "max_length": 50
        },
        {
          "name": "position",
          "type": "Integer",
          "default": 0
        },
        {
          "name": "created_at",
          "type": "DateTime",
          "default": "now()",
          "nullable": false
        },
        {
          "name": "updated_at",
          "type": "DateTime",
          "default": "now()",
          "on_update": "now()",
          "nullable": false
        }
      ],
      "indexes": [
        {
          "name": "idx_category_user",
          "fields": ["user_id", "name"],
          "type": "btree",
          "unique": true
        }
      ],
      "relationships": [
        {
          "type": "many_to_one",
          "target": "User",
          "foreign_key": "user_id",
          "on_delete": "CASCADE"
        }
      ]
    },
    {
      "name": "Comment",
      "table_type": "sql",
      "description": "Task comments for collaboration and notes",
      "fields": [
        {
          "name": "id",
          "type": "UUID",
          "primary_key": true,
          "default": "gen_random_uuid()"
        },
        {
          "name": "task_id",
          "type": "UUID",
          "nullable": false,
          "indexed": true
        },
        {
          "name": "user_id",
          "type": "UUID",
          "nullable": false,
          "indexed": true
        },
        {
          "name": "content",
          "type": "Text",
          "nullable": false
        },
        {
          "name": "attachments",
          "type": "JSON",
          "default": "[]",
          "description": "Array of attachment URLs"
        },
        {
          "name": "created_at",
          "type": "DateTime",
          "default": "now()",
          "nullable": false
        },
        {
          "name": "updated_at",
          "type": "DateTime",
          "default": "now()",
          "on_update": "now()",
          "nullable": false
        },
        {
          "name": "deleted_at",
          "type": "DateTime",
          "nullable": true
        }
      ],
      "indexes": [
        {
          "name": "idx_comment_task",
          "fields": ["task_id", "created_at"],
          "type": "btree"
        },
        {
          "name": "idx_comment_user",
          "fields": ["user_id", "created_at"],
          "type": "btree"
        }
      ],
      "relationships": [
        {
          "type": "many_to_one",
          "target": "Task",
          "foreign_key": "task_id",
          "on_delete": "CASCADE"
        },
        {
          "type": "many_to_one",
          "target": "User",
          "foreign_key": "user_id",
          "on_delete": "CASCADE"
        }
      ]
    },
    {
      "name": "ActivityLog",
      "table_type": "sql",
      "description": "Audit trail for task changes and user actions (admin panel feature)",
      "fields": [
        {
          "name": "id",
          "type": "UUID",
          "primary_key": true,
          "default": "gen_random_uuid()"
        },
        {
          "name": "user_id",
          "type": "UUID",
          "nullable": false,
          "indexed": true
        },
        {
          "name": "entity_type",
          "type": "Enum",
          "values": ["task", "category", "comment", "user"],
          "nullable": false
        },
        {
          "name": "entity_id",
          "type": "UUID",
          "nullable": false,
          "
