### Bluetooth headphones cannot connect
```bash
sudo systemctl restart bluetooth
```
### See the list of the ports that Python scripts created them.
```bash
sudo lsof -n -P -i | grep python
```
Or for a single port:
```bash
sudo lsof -i :<port_number>
```
