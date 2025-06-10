Disable Mac learning on an EX series switch

![image.png](attachment:5e202c32-756d-49f7-a79b-b7909c7a3fe0:image.png)

Extend the mac table aging 

![image.png](attachment:7f9e35a8-00d4-4e0e-8f26-4605289ae6d5:image.png)

Enabling L2 functionality

![image.png](attachment:df64f365-39e8-413a-b919-ef94842dde23:image.png)

```jsx
> show interfaces terse
> show interfaces extensive <interface>
> show ethernet-switching table [extensive]

# identify l2 forwarding table
> show route forwarding-table family ethernet-switching

# clear the mac address table
> clear ethernet-switching table [?]
```

Defining static mac address

![image.png](attachment:7208695f-281b-4f0a-8447-a3238f5e7032:image.png)

```jsx
// load a configuration from a specific directory

edit
> load overide traw/var/hello.config

// save the current config

> save <location>
```

Deleting interfaces using regex

```jsx
# wildcard delete ge-*
```

Interfaces range

![image.png](attachment:88b9d78f-9c5f-46bf-a571-999826c6adda:image.png)

Restart interfaces

![image.png](attachment:c00099e3-4a50-4e64-b535-f073a7634ab9:image.png)

clear ethernet-swtiching table

![image.png](attachment:5733b74f-0bca-472e-9643-8a8b69af5e26:image.png)

set static mac addres for each interface in the default vlan

![image.png](attachment:62550873-3d93-4398-8327-6fca5312f779:image.png)

configure voice vlan

![image.png](attachment:1153e0d3-7083-432e-849b-a43e96750246:image.png)

forwarding-class

![image.png](attachment:247d83f6-7518-44dd-ad1b-e73b5091d814:image.png)

add voice vlan to the trunk port

![image.png](attachment:edb92a2b-d392-457b-ac7b-32e22c7852f7:image.png)

show interfaces terse irb

![image.png](attachment:ce10372e-faee-45b4-baec-ca2e69fe5a66:image.png)