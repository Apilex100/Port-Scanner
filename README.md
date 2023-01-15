
# Port Scanner

Portmapper is python based port and services scanner It can be used with a variety of options Speed of scanning can also be controlled and it is lot faster than conventional tools like NMAP.


# Installation
You need to have python3 to run this on Windows, Linux or MacOS
## Linux
```
git clone https://github.com/Apilex100/Port-Scanner.git
cd Port-Scanner/
python ps.py
```
## Windows
```
git clone https://github.com/Apilex100/Port-Scanner.git
 python .\Port-Scanner\ps.py
```
## Run
### Usage
```
python ps.py -h
usage: python ps.py 192.168.137.52

Python Based Fast Port Scanner

positional arguments:
  IPv4               host to scan

options:
  -h, --help         show this help message and exit
  -s, --start    starting port
  -e, --end      ending port
  -t, --threads  threads to use
  -V, --verbose      verbose output
  -v, --version      display version

Example - ps.py -s 20 -e 40000 -t 500 -V 192.168.137.52
```
