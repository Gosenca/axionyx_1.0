
# File Nyx\_sources\_3d.f90

[**File List**](files.md) **>** [**Source**](dir_74389ed8173ad57b461b9d623a1f3867.md) **>** [**SourceTerms**](dir_7c1c0d2e2a0285e12a54f57a60f809aa.md) **>** [**Nyx\_sources\_3d.f90**](Nyx__sources__3d_8f90.md)

[Go to the documentation of this file.](Nyx__sources__3d_8f90.md) 


````cpp
! :::
! ::: ------------------------------------------------------------------
! :::

      subroutine time_center_sources(lo,hi,state,u_l1,u_l2,u_l3,u_h1,u_h2,u_h3, &
                                     src_old, so_l1,so_l2,so_l3,so_h1,so_h2,so_h3, &
                                     src_new, sn_l1,sn_l2,sn_l3,sn_h1,sn_h2,sn_h3, &
                                     a_old, a_new, dt, print_fortran_warnings) &
                                     bind(C, name="time_center_sources")

      use amrex_error_module, only : amrex_abort
      use amrex_fort_module, only : rt => amrex_real
      use meth_params_module, only : nvar, urho, umx, umz, ueden, ueint
      use  eos_params_module
      use eos_module
      use network

      implicit none

      integer          :: lo(3), hi(3)
      integer          ::  u_l1, u_l2, u_l3, u_h1, u_h2, u_h3
      integer          :: so_l1,so_l2,so_l3,so_h1,so_h2,so_h3
      integer          :: sn_l1,sn_l2,sn_l3,sn_h1,sn_h2,sn_h3
      integer          :: print_fortran_warnings
      real(rt) ::   state( u_l1: u_h1, u_l2: u_h2, u_l3: u_h3,NVAR)
      real(rt) :: src_old(so_l1:so_h1,so_l2:so_h2,so_l3:so_h3,NVAR)
      real(rt) :: src_new(sn_l1:sn_h1,sn_l2:sn_h2,sn_l3:sn_h3,NVAR)
      real(rt) :: a_old, a_new, dt

      ! Local variables
      integer          :: i,j,k,n
      real(rt) :: a_half, a_newsq

      a_half  = 0.5d0 * (a_old + a_new)
      a_newsq = a_new * a_new

      do n = 1, nvar
      do k = lo(3),hi(3)
      do j = lo(2),hi(2)
      do i = lo(1),hi(1)

         ! Density
         if (n.eq.urho) then
            state(i,j,k,n) = state(i,j,k,n) + 0.5d0 * dt * (src_new(i,j,k,n) - src_old(i,j,k,n)) / a_half

         ! Momentum
         else if (n.ge.umx .and. n.le.umz) then
            state(i,j,k,n) = state(i,j,k,n) + 0.5d0 * dt * (src_new(i,j,k,n) - src_old(i,j,k,n)) / a_new

         ! (rho e) and (rho E)
         else if (n.eq.ueint) then

            if (state(i,j,k,ueint) .lt. 0.d0 .and. print_fortran_warnings .gt. 0) then
               print *,'(rho e) negative from old src in time_center_sources: ',i,j,k
            end if

            state(i,j,k,ueint) = state(i,j,k,ueint) + &
                0.5d0 * dt * a_half * (src_new(i,j,k,ueint) - src_old(i,j,k,ueint)) / a_newsq

            state(i,j,k,ueden) = state(i,j,k,ueden) + &
                0.5d0 * dt * a_half * (src_new(i,j,k,ueden) - src_old(i,j,k,ueden)) / a_newsq

            if (state(i,j,k,ueint) .lt. 0.d0 .and. print_fortran_warnings .gt. 0) then
               print *,'(rho e) going negative in time_center_sources: ',i,j,k,state(i,j,k,ueden)
               call amrex_abort("time_center_sources")
               ! print *,' ... resetting to small_temp '
               ! ke = state(i,j,k,UEDEN) - state(i,j,k,UEINT)
               ! call nyx_eos_given_RT(eint, dummy_pres, state(i,j,k,URHO), small_temp, &
               !                      d(i,j,k,NE_COMP),a_new)
               ! state(i,j,k,UEINT) = eint * state(i,j,k,URHO)
               ! state(i,j,k,UEDEN) = state(i,j,k,UEINT) + ke
            end if

         ! Don't do anything here because we have already dealt with UEDEN
         else if (n.eq.ueden) then

         ! (rho X_i) and (rho adv_i) and (rho aux_i)
         else
            state(i,j,k,n) = state(i,j,k,n) + 0.5d0 * dt * (src_new(i,j,k,n) - src_old(i,j,k,n)) / a_half

         end if

      end do
      end do
      end do
      end do

      end subroutine time_center_sources

````

