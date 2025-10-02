---
layout: default
title: Day 0 - Development Environment Setup
nav_order: 1
---


# Day 0: Development Environment Setup

## Overview
Before we start building our full-stack application, we need to set up our development environment. This includes installing the necessary tools and configuring them properly. Follow the manual installation steps below.

## Windows Users: WSL Setup

### What is WSL?
WSL (Windows Subsystem for Linux) allows you to run a Linux environment directly on Windows. This is recommended for web development as it provides a more consistent experience.

### Installing WSL
1. **Open PowerShell as Administrator**
   - Press `Windows + X`
   - Select "Windows PowerShell (Admin)"

2. **Install WSL**
   ```powershell
   wsl --install
   ```

3. **Restart your computer** when prompted

4. **Set up Ubuntu**
   - After restart, Ubuntu will launch automatically
   - Create a username and password when prompted

5. **Update Ubuntu**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

### Alternative: Using Windows Terminal
1. Install Windows Terminal from Microsoft Store
2. Open Windows Terminal
3. Select Ubuntu from the dropdown

![Select Ubuntu Terminal](/bootcamp/assets/images/selecting_ubantu_terminal.png)


## Linux/macOS Users
If you're already on Linux or macOS, you can skip the WSL section and proceed directly to the software installation.

## Software Installation

### 1. Python Installation

#### Check if Python is already installed
```bash
python3 --version
```

#### Install Python (if not installed)

##### **Ubuntu/Debian/WSL**

#### Create a virtual environment (best practice)
```bash
# Create a directory for your projects
mkdir ~/bootcamp-projects
cd ~/bootcamp-projects

# Create a virtual environment
python3 -m venv bootcamp-env

# Activate the virtual environment
source bootcamp-env/bin/activate

# Verify activation (you should see (bootcamp-env) in your prompt)
which python
```

### 2. Node.js and npm Installation

#### Install Node.js using Node Version Manager (nvm)
```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Reload your shell
source ~/.bashrc

# Install the latest LTS version of Node.js
nvm install --lts
nvm use --lts

# Verify installation
node --version
npm --version
```

#### Alternative: Direct installation
```bash
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# macOS (using Homebrew)
brew install node
```

### 3. Git Setup

#### Install Git
```bash
# Ubuntu/Debian
sudo apt install git -y

# macOS (using Homebrew)
brew install git

# Verify installation
git --version
```

#### Configure Git
```bash
# Set your name and email
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Verify configuration
git config --list
```

#### Generate SSH Key for GitHub
```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"
# Plan of Action

# Press Enter to accept default file location
# Enter a passphrase / or click enter to skip (optional)
```
![SSH Keygen Screenshot](/bootcamp/assets/images/ssh_keygen.png)

```bash
# Display public key
cat ~/.ssh/id_ed25519.pub
```

![SSH Keygen Screenshot](/bootcamp/assets/images/public_key_ssh.png)



#### Add SSH Key to GitHub
1. Copy the output from the previous command
2. Go to GitHub.com and sign in
3. Click your profile picture → Settings
4. Click "SSH and GPG keys" in the left sidebar
5. Click "New SSH key"
6. Paste your public key and give it a title
7. Click "Add SSH key"

![Add SSH Key to GitHub](/bootcamp/assets/images/add_ssh_key_github.png)

#### Test SSH Connection
```bash
ssh -T git@github.com
```

![Add SSH Key to GitHub](/bootcamp/assets/images/ssh_verification.png)

## Creating Your First Vue.js Project

### 1. Create a New Vue Project
```bash
# Navigate to your projects directory
cd ~/bootcamp-projects

# Create a new Vue project with Vite
npm create vue@latest my-first-vue-app

# Follow the prompts:
# - Project name: my-first-vue-app
# - Add TypeScript? No
# - Add JSX Support? No
# - Add Vue Router? Yes
# - Add Pinia? No
# - Add Vitest? No
# - Add E2E Testing? No
# - Add ESLint? Yes
# - Add Prettier? Yes
```

### 2. Navigate to Project and Install Dependencies
```bash
cd my-first-vue-app
npm install
```

### 3. Start Development Server
```bash
npm run dev
```

### 4. View Your Application
- Open your browser and go to `http://localhost:5173`
- You should see the Vue.js welcome page

### 5. Stop the Development Server
- Press `Ctrl + C` in the terminal to stop the server

## Setting Up Your Bootcamp Repository

### 1. Create GitHub Repository
1. Go to GitHub.com and sign in
2. Click the "+" icon in the top right
3. Select "New repository"
4. Name it "fullstack-bootcamp"
5. Add description: "Full-stack bootcamp project with Flask and Vue.js"
6. Make it public
7. Don't initialize with README (we'll do this locally)
8. Click "Create repository"

### 2. Initialize Local Repository
```bash
# Navigate to your projects directory
cd ~/bootcamp-projects

# Create bootcamp directory
mkdir fullstack-bootcamp
cd fullstack-bootcamp

# Initialize git repository
git init

# Create initial files
echo "# Full-Stack Bootcamp Project" > README.md
echo "This repository contains my full-stack bootcamp project." >> README.md

# Add files to git
git add README.md

# Make initial commit
git commit -m "Initial commit: Add README"

# Add remote repository
git remote add origin git@github.com:YOUR_USERNAME/fullstack-bootcamp.git

# Push to GitHub
git push -u origin main
```

## Development Workflow Setup

### 1. Create Project Structure
```bash
# Create basic project structure
mkdir backend frontend
touch backend/.gitkeep frontend/.gitkeep

# Create .gitignore file
cat > .gitignore << EOF
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/
env.bak/
venv.bak/
.pytest_cache/

# Node.js
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.npm
.yarn-integrity

# IDEs
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Environment variables
.env

# Database
*.db
*.sqlite3

# Logs
logs/
*.log
EOF

# Add and commit structure
git add .
git commit -m "Add project structure and gitignore"
git push
```

##  Official Documentation & References

Please check the following official documents and resources for detailed guidance:

### Git & GitHub

- **Git Tracker Documentation:**  
  [Google Docs - Git Tracker](https://docs.google.com/document/u/2/d/e/2PACX-1vSQb6y00ELBQ4MC1FAhn1JO8LmVsxGe1ke_aJi8hEIEHYLO8RqHDzl0bxZtxj6J-AYumNrA7jdxRmyu/pub)

- **Git & GitHub Tutorial:**  
  [GitHub Tutorial (GitHub Repo)](https://github.com/shrikrishna97/Resources-App-Dev/blob/main/github_tutorial.md)

- **Adding Git Collaborators:**  
  [How to Add Collaborators (GitHub Tutorial)](https://github.com/shrikrishna97/Resources-App-Dev/blob/main/github_tutorial.md)

---

###  Problem Statements

- **Hospital Management System:**  
  [Hospital Management System Problem Statement](https://docs.google.com/document/d/e/2PACX-1vT24eOovDRvuEmX40yLNycO5qWIxyzPAY00yenNW83nDeEELuZd5YJ47JDhpnYNL1sxnYDkBFTBY9My/pub)

- **Vehicle Parking System:**  
  [Vehicle Parking Problem Statement](https://docs.google.com/document/d/e/2PACX-1vQdJBJIK0445RS8pl6sAX1pH9hSSWB9NCKfC0jx_9QqKt4frI43HeEvlYQYi1N9HkZ827Z3n4_BiJ8n/pub)

---



**You are all set!**

Your development environment is ready. You have:
- Installed Python, Node.js, and Git
- Configured GitHub

You’re ready to start building full stack applications!

---

**What’s Next?**

In next session, we’ll begin Day 1 of the bootcamp:
- Create your first Flask application
- Set up a database
- Start building your first project

If you have any issues, just ask for help in the google space.


