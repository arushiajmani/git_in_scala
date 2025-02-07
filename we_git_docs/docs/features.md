# **WeGit Features**
## Core Components

### 1. Object Storage System

```python
class GitObject:
    def __init__(self, type_name, data):
        self.type = type_name
        self.data = data
        self.hash = self._calculate_hash()

    def _calculate_hash(self):
        """Calculate SHA-1 hash of content with header"""
        header = f"{self.type} {len(self.data)}\0"
        content = header.encode() + self.data
        return hashlib.sha1(content).hexdigest()

    def compress(self):
        """Compress object data using zlib"""
        return zlib.compress(self.data)

    @staticmethod
    def decompress(data):
        """Decompress object data"""
        return zlib.decompress(data)
```

### 2. Index Management

```python
class GitIndex:
    def __init__(self, repo_path):
        self.index_path = os.path.join(repo_path, '.git', 'index')
        self.entries = {}

    def add_entry(self, path, hash_value, stats):
        """Add or update an entry in the index"""
        self.entries[path] = {
            'hash': hash_value,
            'mode': stats.st_mode,
            'size': stats.st_size,
            'mtime': int(stats.st_mtime),
            'ctime': int(stats.st_ctime)
        }

    def remove_entry(self, path):
        """Remove an entry from the index"""
        if path in self.entries:
            del self.entries[path]

    def write(self):
        """Write index to disk"""
        # Implementation of index serialization
```

## Command Implementations

### 1. Init Command

```python
class InitCommand:
    def execute(self, path="."):
        """Initialize a new Git repository"""
        git_dir = os.path.join(path, '.git')
        
        # Create directory structure
        os.makedirs(git_dir)
        os.makedirs(os.path.join(git_dir, 'objects'))
        os.makedirs(os.path.join(git_dir, 'refs', 'heads'))
        
        # Create HEAD file
        with open(os.path.join(git_dir, 'HEAD'), 'w') as f:
            f.write('ref: refs/heads/master\n')
        
        print(f"Initialized empty Git repository in {git_dir}")
```

### 2. Add Command

```python
class AddCommand:
    def __init__(self, repo):
        self.repo = repo
        self.index = repo.index

    def add_file(self, path):
        """Add a single file to the staging area"""
        with open(path, 'rb') as f:
            data = f.read()
        
        # Create blob object
        blob = GitObject('blob', data)
        
        # Store compressed object
        self.repo.store_object(blob)
        
        # Update index
        stats = os.stat(path)
        self.index.add_entry(path, blob.hash, stats)

    def add_all(self):
        """Add all changes in the working directory"""
        for root, _, files in os.walk('.'):
            for file in files:
                if not file.startswith('.git'):
                    path = os.path.join(root, file)
                    self.add_file(path)
```

### 3. Restore Command

```python
class RestoreCommand:
    def __init__(self, repo):
        self.repo = repo

    def restore_file(self, path):
        """Restore file to its state in HEAD"""
        # Get file hash from HEAD
        head_tree = self.repo.get_head_tree()
        file_hash = head_tree.get_hash_for_path(path)

        if file_hash:
            # Retrieve and decompress object
            blob = self.repo.get_object(file_hash)
            data = GitObject.decompress(blob.data)

            # Write file
            with open(path, 'wb') as f:
                f.write(data)
            
            print(f"Restored {path}")
        else:
            print(f"File {path} not found in HEAD")
```

### 4. Remove Command

```python
class RemoveCommand:
    def __init__(self, repo):
        self.repo = repo
        self.index = repo.index

    def remove(self, path, cached=False):
        """Remove file from working directory and/or index"""
        if not cached:
            # Remove from working directory
            try:
                os.remove(path)
            except FileNotFoundError:
                pass

        # Remove from index
        self.index.remove_entry(path)
        self.index.write()
```

## Helper Functions

### Hash Utilities

```python
def create_hash_path(hash_value):
    """Create path for object storage based on hash"""
    return os.path.join(
        '.git/objects',
        hash_value[:2],
        hash_value[2:]
    )

def hash_file(path):
    """Calculate hash of file content"""
    with open(path, 'rb') as f:
        data = f.read()
    return hashlib.sha1(data).hexdigest()
```

### File System Operations

```python
def ensure_dir_exists(path):
    """Ensure directory exists, create if necessary"""
    os.makedirs(path, exist_ok=True)

def is_ignored(path):
    """Check if path should be ignored"""
    ignore_patterns = ['.git', '__pycache__', '*.pyc']
    return any(fnmatch.fnmatch(path, pattern) for pattern in ignore_patterns)
```

## Usage Examples

### Basic Workflow

```python
# Initialize repository
repo = Repository('.')
init_cmd = InitCommand()
init_cmd.execute()

# Add files
add_cmd = AddCommand(repo)
add_cmd.add_file('example.txt')

# Remove file
rm_cmd = RemoveCommand(repo)
rm_cmd.remove('old_file.txt', cached=True)

# Restore file
restore_cmd = RestoreCommand(repo)
restore_cmd.restore_file('modified_file.txt')
```