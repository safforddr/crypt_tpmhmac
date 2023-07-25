# crypt_tpmhmac
Use a TPM based hmac for shadow passwords

These patches to libxcrypt and pam provide a shadow password hash
that cannot be compromised in an offline attack. A local hardware
TPM is used to HMAC the salt and passphrase to generate the hash.
Since am attacker cannot obtain the TPM's HMAC key, there is no
way to mount any offline attacks. Details are provided in tpmhmac.doc
These patches were developed on Fedora 38. YMMV.

Installation:
1. Apply the supplied patches to libxcrypt and pam source, compile and install.
   (On fedora I used the respective source rpms, adding the patches to rpmbuild/SOURCES, and spec files.)
2. copy tpmhmac.conf to /etc
3. Provision your TPM as with an SRK and optionally a DRSK as described in the TPM_KEYS repository.
4. create the HMAC keys. For example, with a DRSK at 0x81000004
      tpm2_create -C 0x81000004 -G hmac -a "sensitivedataorigin|userwithauth|sign" -u hmac.pub -r hmac.priv
   and place them in /etc.
   (The parent key handle and hmac key filenames can be tailored in /etc/tpmhmac.conf.)
5. in /etc/login.defs change ENCRYPT_METHOD to
        ENCRYPT_METHOD TPMHMAC
       
All new password hashes will now use the TPM. Existing hashes continue to use their existing hash
even when updated with passwd. To force en existing hash to change to tpmhmac, you have to delete
the hash for that account in /etc/shadow (everything between the first and second ':'.)
