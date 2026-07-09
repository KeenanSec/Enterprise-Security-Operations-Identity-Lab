## Create a Windows Server VM

1. First I downloaded Windows Server 2022

```
wget https://software-static.download.prss.microsoft.com/sg/download/888969d5-f34g-4e03-ac9d-1f9786c66749/SERVER_EVAL_x64FRE_en-us.iso
```

![Pasted image 20260625042343](images/Pasted%20image%2020260625042343.png)

![Pasted image 20260625043240](images/Pasted%20image%2020260625043240.png)

![Pasted image 20260625043740](images/Pasted%20image%2020260625043740.png)

![Pasted image 20260625044153](images/Pasted%20image%2020260625044153.png)


# Setup Active Directory

2. Click `Add Roles and Features`

![Pasted image 20260625045552](images/Pasted%20image%2020260625045552.png)

3. Click `Next` 
<img src="images/Pasted%20image%2020260625045714.png" alt="Pasted image 20260625045714" width="448">

4. Click `Next` for `Installation Type

![Pasted image 20260625045802](images/Pasted%20image%2020260625045802.png)

5. Choose the server from the server pool & click `Next`

![Pasted image 20260625045839](images/Pasted%20image%2020260625045839.png)

6. Check the box for `Active Directory Domain Services`

![Pasted image 20260625050041](images/Pasted%20image%2020260625050041.png)

7. Select `Add Features`

![Pasted image 20260625050129](images/Pasted%20image%2020260625050129.png)

8. Select `Next`

![Pasted image 20260625050239](images/Pasted%20image%2020260625050239.png)

9. Select `Next` on the `Active Directory Domain Services`

![Pasted image 20260625050309](images/Pasted%20image%2020260625050309.png)


10. Then Click `Install` on the Confirmation page.

![Pasted image 20260625050427](images/Pasted%20image%2020260625050427.png)


11. Click on the `Yellow Triangle` next to the Flag on the top right. Then Click `Promote tjos server to a domain controller`

![Pasted image 20260625050718](images/Pasted%20image%2020260625050718.png)

12. Then select `Add a new forest` & add the domain name you want.

![Pasted image 20260625051049](images/Pasted%20image%2020260625051049.png)

13. Then Create a `Directory Services Restore Mode Password`.  & click `Next`.

![Pasted image 20260625051259](images/Pasted%20image%2020260625051259.png)

14. Click `Next`

![Pasted image 20260625051353](images/Pasted%20image%2020260625051353.png)

15. Then wait for the default NetBIOS name & then click `Next`.

![Pasted image 20260625051428](images/Pasted%20image%2020260625051428.png)

16. Select `Next` again & leave the default paths alone.

![Pasted image 20260625051523](images/Pasted%20image%2020260625051523.png)

17. Select `Next` if everything is correct.

![Pasted image 20260625051624](images/Pasted%20image%2020260625051624.png)

18. Then click `Install`

![Pasted image 20260625052854](images/Pasted%20image%2020260625052854.png)



## References

```
https://www.ibm.com/docs/en/storage-scale-bda?topic=support-install-configure-active-directory
```