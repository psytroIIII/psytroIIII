# pragma une fois
# inclure  " ../ironsight/sdk.h "
extern  bool g_aim_key_down;

espace de noms  aimbot
{
	statique D3DXVECTOR2 calculate_angles ( const CActor* plocal_actor, const CActor* ptarget )
	{
		const  auto distance_x = plocal_actor-> m_head_coords . get_distance_x ( ptarget-> m_head_coords );
		const  auto distance_y = plocal_actor-> m_head_coords . get_distance_y ( ptarget-> m_head_coords );
		const  auto distance_z = plocal_actor-> m_head_coords . get_distance_z ( ptarget-> m_head_coords );
		const  auto distance = plocal_actor-> m_head_coords . get_3d_distance ( ptarget-> m_head_coords );

		D3DXVECTOR2 angle_cible = { 0 , 0 };

		if ( ptarget-> m_head_coords . x < plocal_actor-> m_head_coords . x &&
			 ptarget-> m_head_coords . z > plocal_actor-> m_head_coords . z )
		{
			angle_cible. y = ( atanf ( abs ( distance_x / distance_z ) ) / PI ) * 180 . 0f ;
		}

		else  if ( ptarget-> m_head_coords . x > plocal_actor-> m_head_coords . x &&
			      ptarget-> m_head_coords . z > plocal_actor-> m_head_coords . z )
		{
			angle_cible. y = ( ( atanf ( abs ( distance_x / distance_z ) ) / PI ) * 180 . 0f ) * - 1 . 0f ;
		}

		else  if ( ptarget-> m_head_coords . x > plocal_actor-> m_head_coords . x &&
			      ptarget-> m_head_coords . z < plocal_actor-> m_head_coords . z )
		{
			angle_cible. y = ( ( atanf ( abs ( distance_x / distance_z ) ) / PI ) * 180 . 0f ) + 180 . 0f ;
		}

		else  if ( ptarget-> m_head_coords . x < plocal_actor-> m_head_coords . x &&
			      ptarget-> m_head_coords . z < plocal_actor-> m_head_coords . z )
		{
			angle_cible. y = 180 . 0f - ( ( atanf ( abs ( distance_x / distance_z ) ) / PI ) * 180 . 0f );
		}

		angle_cible. x = ( ( - asinf ( distance_y / distance ) / PI ) * 180 . 0f ) - 0 . 2f ;

		return target_angle;
	}
	
	static  bool  get_nearest_by_fov ( float fov, const D3DXVECTOR2 view_angles, const D3DXVECTOR2 target_angles )
	{
		fov /= 2 . 25f ;
		
		renvoie target_angles. y >= ( angles_vue. y - fov ) &&
			   angles_cibles. y <= ( angles_vue. y + fov ) &&
			   angles_cibles. x >= ( angles_vue. x - fov ) &&
			   angles_cibles. x <= ( angles_vue. x + fov );
	}
	
	CActor statique * get_nearest_entity ( const CActor * plocal_actor )
	{	
		si ( !plocal_actor )
			return  nullptr ;

		CActor* pnearest_entity = nullptr ;
		distance_la plus basse automatique = 999999 . 0f ;

		// g_entity_manager 8B 0D ? ? ? ? 85 C9 74 15 83 C8 FF F0 0F C1 41 ? 48 75 0A 85 C9 74 06 8B 01 6A 01 FF 10 E8 ? ? ? ?
		const  auto pentity_mgr = * reinterpret_cast <CEntityManager **> (+ ironsight_base 0xA88B30 );

		si ( pentity_mgr )
		{
			__essayer
			{
				const  auto pentity_list = pentity_mgr-> m_list ;

				if ( liste_pentity )
				{
					auto pentity_node = pentity_list-> m_head ;

					if ( pentity_node )
					{
						pour (;! pentity_node = reinterpret_cast <CEntityNode *> (pentity_list);)
						{
							auto pentity = pentity_node-> m_instance ;

							if ( pentity != plocal_actor && pentity-> m_vtable_ptr == plocal_actor-> m_vtable_ptr &&
								 pentity-> is_alive () && pentity-> m_teamid != plocal_actor-> m_teamid )
							{

								if ( config::b_nearest_by_fov &&
									 get_nearest_by_fov ( static_cast < float >( config::i_fov ), plocal_actor-> m_view_angles , calculate_angles ( plocal_actor, pentity ) ) )
								{
									pnearest_entity = repentir;
									return pnearest_entity;
								}

								else  if ( !config::b_nearest_by_fov )
								{
									const  auto distance = plocal_actor-> m_head_coords . get_3d_distance ( pentity-> m_head_coords );

									if ( distance < distance_la plus basse )
									{
										distance_la plus basse = distance ;
										pnearest_entity = repentir;
									}
								}
							}

							pentity_node = pentity_node-> m_next ;
						}
					}
				}
			}

			__sauf ( EXCEPTION_EXECUTE_HANDLER )
			{
				return  nullptr ;
			}
		}

		return pnearest_entity;
	}

	static  void  aimbot () noexcept
	{
		si ( g_aim_key_down )
		{
			// g_local_actor -> g_entity_manager + 0x4
			const  auto plocal_actor = * reinterpret_cast <CActor **> (+ ironsight_base 0xA88B34 );
			const  auto ptarget = get_nearest_entity ( plocal_actor );

			si ( !pcible )
				retour ;

			const  auto target_angle = calculate_angles ( plocal_actor, ptarget );
			
			si ( config::b_smooth )
			{
				plocal_actor-> m_view_angles . x += ( target_angle. x - plocal_actor-> m_view_angles . x ) / static_cast < float >( config::i_smooth );
				plocal_actor-> m_view_angles . y += ( target_angle. y - plocal_actor-> m_view_angles . y ) / static_cast < float >( config::i_smooth );
				retour ;
			}

			plocal_actor-> m_view_angles . x = angle_cible. x ;
			plocal_actor-> m_view_angles . y = angle_cible. y ;
		}
	}

	static  void  fov ( const ImVec2 position, const  int radius, float * color ) noexcept
	{
		render::draw_circle ( position, static_cast < float >( radius ), color );
	}
}
