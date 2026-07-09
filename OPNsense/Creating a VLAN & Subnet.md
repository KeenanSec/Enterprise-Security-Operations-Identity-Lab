
Now I need to create a VLAN interface 

First I went to `Interfaces` > Devices > VLAN

![Pasted image 20260707192823](Images/Pasted%20image%2020260707192823.png)


When I got there I press the orange `+`


![Pasted image 20260707192958](Images/Pasted%20image%2020260707192958.png)

Then go to Assignments 


![Pasted image 20260707194020](Images/Pasted%20image%2020260707194020.png)

Click on `HOME` and enable the interface

![Pasted image 20260707194218](Images/Pasted%20image%2020260707194218.png)

Then after that scroll down and change the `IPv4 Configuration Type` to `Static IPv4` & then scroll all the way to bottom and I statically assigned the IP to be in the `.60` subnet.

![Pasted image 20260707194418](Images/Pasted%20image%2020260707194418.png)


Select `Save` & then `Apply Changes`


After that go to `Services` on the left panel & I selected `Kea DHCP` > `Kea DHCPv4`

![Pasted image 20260707194654](Images/Pasted%20image%2020260707194654.png)


At the `settings` page I landed I then selected `HOME` so that I can then create a subnet for the `HOME` VLAN Interface.

![Pasted image 20260707194759](Images/Pasted%20image%2020260707194759.png)

So I selected `Subnets` > then clicked the orange `+`. Then I defined the range of IPs I want the VLAN gateway to assign devices.

![Pasted image 20260707195119](Images/Pasted%20image%2020260707195119.png)

Select `Apply`

![Pasted image 20260707195321](Images/Pasted%20image%2020260707195321.png)