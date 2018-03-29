---
title: fortran_string
tags:
---
# Fortran 字符串处理

1. 字符串链接 `//`

``` fortran
character(len=100) x,y,z
x='a'   ! 空格补到100个字符串为止
y='b'   ! 空格补到100个字符串为止
z=x//y  ! 200个字符串已经超出z的定义范围
```
==正确方式==
``` Fortran
character(len=100) x,y,z
x='a'			! 空格补到100个字符串为止
y='b'
z=trim(x)//trim(y) ! trim是去掉字符串尾部的空格
```

2. 字符串模块

``` Fortran
MODULE String_Functions
IMPLICIT NONE
! Copy (generic) char array to string or string to char array
! Clen           returns same as LEN      unless last non-blank char = null
! Clen_trim      returns same as LEN_TRIM    "              "
! Ctrim          returns same as TRIM        "              "
! Count_Items    in string that are blank or comma separated
! Reduce_Blanks  in string to 1 blank between items, last char not blank
! Replace_Text   in all occurances in string with replacement string
! Spack          pack string's chars == extract string's chars
! Tally          occurances in string of text arg
! Translate      text arg via indexed code table
! Upper/Lower    case the text arg

INTERFACE Copy    ! generic
   MODULE PROCEDURE copy_a2s, copy_s2a
END INTERFACE Copy

CONTAINS
! ------------------------
PURE FUNCTION Copy_a2s(a)  RESULT (s)    ! copy char array to string
CHARACTER,INTENT(IN) :: a(:)
CHARACTER(SIZE(a)) :: s
INTEGER :: i
DO i = 1,SIZE(a)
   s(i:i) = a(i)
END DO
END FUNCTION Copy_a2s

!------------------------
PURE FUNCTION Copy_s2a(s)  RESULT (a)   ! copy s(1:Clen(s)) to char array
CHARACTER(*),INTENT(IN) :: s
CHARACTER :: a(LEN(s))
INTEGER :: i
DO i = 1,LEN(s)
   a(i) = s(i:i)
END DO
END FUNCTION Copy_s2a

! ------------------------
PURE INTEGER FUNCTION Clen(s)      ! returns same result as LEN unless:
CHARACTER(*),INTENT(IN) :: s       ! last non-blank char is null
INTEGER :: i
Clen = LEN(s)
i = LEN_TRIM(s)
IF (s(i:i) == CHAR(0)) Clen = i-1  ! len of C string
END FUNCTION Clen

! ------------------------
PURE INTEGER FUNCTION Clen_trim(s) ! returns same result as LEN_TRIM unless:
CHARACTER(*),INTENT(IN) :: s       ! last char non-blank is null, if true:
INTEGER :: i                       ! then len of C string is returned, note:
                                   ! Ctrim is only user of this function
i = LEN_TRIM(s) ; Clen_trim = i
IF (s(i:i) == CHAR(0)) Clen_trim = Clen(s)   ! len of C string
END FUNCTION Clen_trim

! ----------------
FUNCTION Ctrim(s1)  RESULT(s2)     ! returns same result as TRIM unless:
CHARACTER(*),INTENT(IN)  :: s1     ! last non-blank char is null in which
CHARACTER(Clen_trim(s1)) :: s2     ! case trailing blanks prior to null
s2 = s1                            ! are output
END FUNCTION Ctrim

! --------------------
INTEGER FUNCTION Count_Items(s1)  ! in string or C string that are blank or comma separated
CHARACTER(*) :: s1
CHARACTER(Clen(s1)) :: s
INTEGER :: i, k

s = s1                            ! remove possible last char null
k = 0  ; IF (s /= ' ') k = 1      ! string has at least 1 item
DO i = 1,LEN_TRIM(s)-1
   IF (s(i:i) /= ' '.AND.s(i:i) /= ',' &
                    .AND.s(i+1:i+1) == ' '.OR.s(i+1:i+1) == ',') k = k+1
END DO
Count_Items = k
END FUNCTION Count_Items

! --------------------
FUNCTION Reduce_Blanks(s)  RESULT (outs)
CHARACTER(*)      :: s
CHARACTER(LEN_TRIM(s)) :: outs
INTEGER           :: i, k, n

n = 0  ; k = LEN_TRIM(s)          ! k=index last non-blank (may be null)
DO i = 1,k-1                      ! dont process last char yet
   n = n+1 ; outs(n:n) = s(i:i)
   IF (s(i:i+1) == '  ') n = n-1  ! backup/discard consecutive output blank
END DO
n = n+1  ; outs(n:n)  = s(k:k)    ! last non-blank char output (may be null)
IF (n < k) outs(n+1:) = ' '       ! pad trailing blanks
END FUNCTION Reduce_Blanks

! ------------------
FUNCTION Replace_Text (s,text,rep)  RESULT(outs)
CHARACTER(*)        :: s,text,rep
CHARACTER(LEN(s)+100) :: outs     ! provide outs with extra 100 char len
INTEGER             :: i, nt, nr

outs = s ; nt = LEN_TRIM(text) ; nr = LEN_TRIM(rep)
DO
   i = INDEX(outs,text(:nt)) ; IF (i == 0) EXIT
   outs = outs(:i-1) // rep(:nr) // outs(i+nt:)
END DO
END FUNCTION Replace_Text

! ---------------------------------
FUNCTION Spack (s,ex)  RESULT (outs)
CHARACTER(*) :: s,ex
CHARACTER(LEN(s)) :: outs
CHARACTER :: aex(LEN(ex))   ! array of ex chars to extract
INTEGER   :: i, n

n = 0  ;  aex = Copy(ex)
DO i = 1,LEN(s)
   IF (.NOT.ANY(s(i:i) == aex)) CYCLE   ! dont pack char
   n = n+1 ; outs(n:n) = s(i:i)
END DO
outs(n+1:) = ' '     ! pad with trailing blanks
END FUNCTION Spack

! --------------------
INTEGER FUNCTION Tally (s,text)
CHARACTER(*) :: s, text
INTEGER :: i, nt

Tally = 0 ; nt = LEN_TRIM(text)
DO i = 1,LEN(s)-nt+1
   IF (s(i:i+nt-1) == text(:nt)) Tally = Tally+1
END DO
END FUNCTION Tally

! ---------------------------------
FUNCTION Translate(s1,codes)  RESULT (s2)
CHARACTER(*)       :: s1, codes(2)
CHARACTER(LEN(s1)) :: s2
CHARACTER          :: ch
INTEGER            :: i, j

DO i = 1,LEN(s1)
   ch = s1(i:i)
   j = INDEX(codes(1),ch) ; IF (j > 0) ch = codes(2)(j:j)
   s2(i:i) = ch
END DO
END FUNCTION Translate

! ---------------------------------
FUNCTION Upper(s1)  RESULT (s2)
CHARACTER(*)       :: s1
CHARACTER(LEN(s1)) :: s2
CHARACTER          :: ch
INTEGER,PARAMETER  :: DUC = ICHAR('A') - ICHAR('a')
INTEGER            :: i

DO i = 1,LEN(s1)
   ch = s1(i:i)
   IF (ch >= 'a'.AND.ch <= 'z') ch = CHAR(ICHAR(ch)+DUC)
   s2(i:i) = ch
END DO
END FUNCTION Upper

! ---------------------------------
FUNCTION Lower(s1)  RESULT (s2)
CHARACTER(*)       :: s1
CHARACTER(LEN(s1)) :: s2
CHARACTER          :: ch
INTEGER,PARAMETER  :: DUC = ICHAR('A') - ICHAR('a')
INTEGER            :: i

DO i = 1,LEN(s1)
   ch = s1(i:i)
   IF (ch >= 'A'.AND.ch <= 'Z') ch = CHAR(ICHAR(ch)-DUC)
   s2(i:i) = ch
END DO
END FUNCTION Lower

END MODULE String_Functions
```

3.  fortran 带命令行参数运行

```
POGRAM test_get_command_argument
INTEGER :: i,n
CHARACTER(len=128) :: arg
i = 0
DO
  CALL get_command_argument(i, arg)
      IF (LEN_TRIM(arg) == 0) EXIT
        WRITE (*,*) i,arg
          i = i+1
END DO
END PROGRAM 



 PROGRAM test_get_command_argument
 INTEGER :: i,n
 CHARACTER(len=128) :: arg
 n = iargc()
    DO i=0,n
      CALL getarg(i, arg)
      WRITE (*,*) i,arg
     END DO
END PROGRAM
```
```

selectcase (var)
    
	case (1)	 ! var =1

	case (:6)	 ! var <=6

	case (2:)	 ! var >= 2
		
	case  (1:5)      ! 1<=var<=5

	case  (1,2,5,6)  !var= 1 or 2 or 5 or 6


    case default

end select

```

>`./main -a 10 -b 10                    `


```
! cmdline.f90 -- simple command-line argument parsing example

program cmdline
  implicit none

  character(len=*), parameter :: version = '1.0'
  character(len=32) :: arg
  character(len=8) :: date
  character(len=10) :: time
  character(len=5) :: zone
  logical :: do_time = .false.
  integer :: i

  do i = 1, command_argument_count()
     call get_command_argument(i, arg)

     select case (arg)
     case ('-v', '--version')
        print '(2a)', 'cmdline version ', version
        stop
     case ('-h', '--help')
        call print_help()
        stop
     case ('-t', '--time')
        do_time = .true.
     case default
        print '(a,a,/)', 'Unrecognized command-line option: ', arg
        call print_help()
        stop
     end select
  end do

  ! Print the date and, optionally, the time
  call date_and_time(DATE=date, TIME=time, ZONE=zone)
  write (*, '(a,"-",a,"-",a)', advance='no') date(1:4), date(5:6), date(7:8)
  if (do_time) then
     write (*, '(x,a,":",a,x,a)') time(1:2), time(3:4), zone
  else
     write (*, '(a)') ''
  end if

contains

  subroutine print_help()
    print '(a)', 'usage: cmdline [OPTIONS]'
    print '(a)', ''
    print '(a)', 'Without further options, cmdline prints the date and exits.'
    print '(a)', ''
    print '(a)', 'cmdline options:'
    print '(a)', ''
    print '(a)', '  -v, --version     print version information and exit'
    print '(a)', '  -h, --help        print usage information and exit'
    print '(a)', '  -t, --time        print time'
  end subroutine print_help

end program cmdline
```
http://jblevins.org/log/cmdline
http://blog.sciencenet.cn/blog-51026-622520.html

















