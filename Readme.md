# vbqmclient 

  START-HISTORY (vbqmclient):
  xxNov23 mab attempt windows port, using c++ builder ver11.3 CE

  This version supports both QMCall and QMCallx (found the problem with returned values in QMCall)
  Use QMGetArg to access returned values from QMcallx calls
  Warning vbqmclient does not maintian a storage are for Getarg parameters for each session. Using Callx will "overwrite" the previous Callx
  parameters regardless of session number.
  
  Also Note I had to add ALIGN2 to the packet structures used when sending / receiving data from the server. Apparently C++ builder takes liberty about how
  structures are aligned, see https://www.geeksforgeeks.org/structure-member-alignment-padding-and-data-packing/
  Basically copied whatever was packed in qmclilib.c
  
  Notes: There seems to be an issue with char vs unsigned char when using the memchr c function (void *memchr(const void *str, int c, size_t n).
  If passing the int c parameter as a char, characters > 127 cannot be found (interpreted as a neg integer?).
  The gcc compiler & linux runtime seems to be fine with it,  not so with C++ Builder / Windows. Need to use unsigned Char.

  For now I have commented out #include "qmdefs.h" and pulled the required
  information from the include file into this copy of qmclient.c
  There is a bunch of stuff in qmdefs.h that will not resolve with c++ builder
  some of the int defines, bigended stuff. Lazy on my part, yes.

  Replaced a few:
      (char*)src = p + 1;
  With
		src = p + 1;
  Where src is defined as type BSTR.
  New verstion of the compilier does not allow type casting on lvalue.
   Apparently we get away with this because qm only supports ansi strings,
     but I see problems down the road.  You have been warned!

  This is a work in progress.......

  Also contained in this folder: vbaTest.xlsm
  Simple vba macro that tests some of the functions of vbqmclient
  Connect
  Open
  Read
  Write
  Call / Callx
  GetArg
  Dcount
  Ins
  Extract

  To run you need to enable macros in execl and edit the Declare statements in UserForm1 code with the location of vbqmclient.dll 

  Big Note: With Debian 11 / Ubuntu 22.04. The new default password encoding is yescrypt.
  The password test in linuxio.c does not support yescrypt, only MD5 and SHA512
  In order for connect to work you will need to edit  /etc/pam.d/common-password:
  And change:
  original:
   password    [success=2 default=ignore]    pam_unix.so obscure use_authtok try_first_pass yescrypt
  new:
   password    [success=2 default=ignore]    pam_unix.so obscure use_authtok try_first_pass sha512crypt
  then reset your password 
