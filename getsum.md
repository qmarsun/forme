Verify with 
SHA256 checksum de5b410dd0813e5904aeed082bfc3d9a8167e0f93b296f52d80bee2dfec9f13d 
or 
GPG signature by 0xf7a49b0ec.
=======================================
Get-FileHash "C:\...\Downloads\msys2-x86_64-20221216.exe"
PS C:\Users\..\Downloads> Get-FileHash "C:\Users\KumarSundaram\Downloads\msys2-x86_64-20221216.exe"

Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
SHA256          DE5B410DD0813E5904AEED082BFC3D9A8167E0F93B296F52D80BEE2DFEC9F13D     

==============================================================================
=======================================
sha256sum /path/to/data.txt > checksum
cat checksum 
86c5ceb27e1bf441130299c0209e5f35b88089f62c06b2b09d65772274f12057 /path/to/data.txt​
