FOR PRIVACY AND CODE PROTECTING REASONS THIS IS A SIMPLIFIED VERSION OF CHANGES AND NEW FEATURES

TASK DATE: 19.10.2017 - FINISHED: 20.10.2017

TASK LEVEL: (EASY)  

TASK SHORT DESCRIPTION: feature/task-1145-add-date-logged

GITHUB REPOSITORY CODE: feature/task-1145-add-date-logged

CHANGES

	IN FILES:
	
	details.php
	
		added code 1 (upgrade):
		
			if (version_compare($old_version, '2.3.83', 'lt')) 
			{
				//Add new field to default_profiles table
				$sql = "ALTER TABLE default_profiles " . 
							"ADD COLUMN deceased_date_logged_offline BIGINT(20) NULL DEFAULT NULL " .
							"AFTER name_updated_by";
				if ( ! $this->db->query($sql)) {
					return false;
				}							
		
		
				//data of new record (Deceased date logged) into default_data_fields table
				$field_values = array (
					'field_name' => 'Deceased date logged',
					'field_slug' => 'deceased_date_logged_offline',
					'field_namespace' => 'users',
					'field_type' => 'datetime',
					'field_data' => serialize(array(
						'use_time' => 'no',
						'start_date' => '-150Y',
						'end_date' => NULL,
						'storage' => 'unix',
						'input_type' => 'dropdown'
					)),
					'view_options' => NULL,
					'is_locked' => 'no'
				);
				if ( ! $this->db->insert('default_data_fields', $field_values))  {
					return false;
				}							
				
				
				//Getting last insert id
				$last_insert_id = $this->db->insert_id();
				
				
				//Getting profile stream
				$profile_stream = $this->streams->streams->get_stream('profiles', 'users');
				
				
				//data of new record into data_field_assignments
				$field_values = array (
					'sort_order' => 53,
					'stream_id' =>  $profile_stream->id,
					'field_id' => $last_insert_id,
					'is_required' => 'no',
					'is_unique' => 'no',
					'instructions' => NULL,
					'field_name' => NULL
				);
				if ( ! $this->db->insert('data_field_assignments', $field_values))  {
					return false;
				}
				
			
				//update fields order 
				if ( ! $this->update_profile_field_ordering())
					return false;

			} //END upgrade version 2.3.83

			return true;
		}
		
		
		ADDED CODE II (install):
			
			array(
					'name'          => 'Deceased date logged',
					'slug'          => 'deceased_date_logged_offline',
					'type'          => 'datetime',
					'extra'         => array(
						'start_date' => '-150Y',
						'use_time'  => 'no',
						'storage'   => 'unix',
						'input_type' => 'dropdown'
					),
					'required'      => false,
				),


			added code 3 (update fields order)	
			
			deceased_date_logged_offline' => 53,
		
		
		
		
			
	form.php
	
		added and changed code 1
		
			<?php foreach($profile_fields as $field): ?>
										
				<?php if($field['field_slug']=='deceased_date_logged_offline') : ?>										
					<div id="deceased_date_logged" class="deceased-form-hide" <?php  echo "style='margin-bottom: " . ($member->active == 3 ? "40" : "0") . "px;'"; ?>>
						<label for="<?php echo $field['field_slug'] ?>">
							<?php echo (lang($field['field_name'])) ? lang($field['field_name']) : $field['field_name'];  ?>
						</label>
						<div class="input gap_bob">
							<?php echo $field['input']; ?>
						</div>
					</div>
				<?php endif; ?>
				
				
				
				<?php if($field['field_slug']=='deceased_date_offline') : ?>
					<div id="deceased_date" class="deceased-form-hide" <?php  echo "style='margin-bottom: " . ($member->active == 3 ? "40" : "0") . "px;'"; ?>>
						<label for="<?php echo $field['field_slug'] ?>">
							<?php echo (lang($field['field_name'])) ? lang($field['field_name']) : $field['field_name'];  ?>
						</label>
						<div class="input gap_bob">
							<?php echo $field['input']; ?>
						</div>
					</div>	
				<?php endif; ?>	
			
			<?php endforeach; ?>
			
			
			
			
	members.php
	
		changed code: 
		
			$profile_validation = $this->streams->streams->validation_array('profiles', 'users', 'edit', array('first_name', 'last_name', 'dob', 'dob_offline', 'deceased_date_logged_offline', 'deceased_date_offline', 'person_types', 'suffix_one', 'suffix_two'), $id);

		added code: 
		
			if($profile_data['deceased_date_logged_offline_month']=='' or $profile_data['deceased_date_logged_offline_day']=='' or $profile_data['deceased_date_logged_offline_year']=='') {
				$profile_update['deceased_date_logged_offline'] = null;
			}
			
			
		added code 2: 
		
		//Set today's date at Deceased date logged if the value is empty 
		if (
				! $values['deceased_date_logged_offline'] > 0 or
				! $values['deceased_date_offline']
			) {
			$values['deceased_date_logged_offline'] = time();
		}
		
		
		
			
	member_edit.js
	
		changed code: 
		
		$('#active').change(function() {
			if ($(this).val() == '3') {
				$('#deceased_date, #deceased_date_logged').show(400);
				$('#deceased_date .gap_bob, #deceased_date_logged .gap_bob').css('position','absolute');
			}
			else
			{
				$('#deceased_date, #deceased_date_logged').hide(400);
			}
			$('#deceased_date chosen-drop').css("zIndex", 1060);
			$('.deceased-form-hide').css('margin-bottom','+=70px');
		});
