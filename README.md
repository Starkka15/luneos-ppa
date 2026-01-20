# LuneOS PPA for Ubuntu

Package repository for LuneOS on Ubuntu 24.04 (Noble).

## Installation

Add the PPA to your system:

```bash
# Add the repository
echo "deb [trusted=yes] https://starkka15.github.io/luneos-ppa noble main" | sudo tee /etc/apt/sources.list.d/luneos.list

# Update package list
sudo apt update

# Install LuneOS meta package (installs all components)
sudo apt install luneos-core
```

## Manual Package Installation

To install individual packages:

```bash
sudo apt install <package-name>
```

## Available Packages

This PPA contains 213 LuneOS packages including:

- Luna Surface Manager (Wayland compositor)
- Luna System Manager
- Luna App Manager
- webOS services and applications
- LuneOS applications

## Requirements

- Ubuntu 24.04 (Noble Numbat)
- amd64 architecture

## License

Various licenses apply to individual packages. See each package for details.
