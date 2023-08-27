Component: UserProfileEditor

VARIABLES:
  - user data (from firebase)
  - page data (from app stores)

FUNCTIONS:

  - toggleProfileStatus():
      SET userRef to point to current user's document in the "users" collection in the database
      UPDATE the "published" field in userRef with the opposite of its current value
      

HTML STRUCTURE:

  CHECK if user data's username matches the page's username parameter:
    - DISPLAY heading "Edit your Profile"
    - DISPLAY the profile link in format: https://fuguely-web-app.web.app/{username}
    - BUTTONS to:
      - Change photo
      - Edit bio
      
    - FORM for toggling profile visibility:
      - TOOLTIP explaining the purpose of the visibility toggle
      - TOGGLE SWITCH:
        - If user data's published field is true, show as "Public profile"
        - If user data's published field is false, show as "Private profile"
        - ON CHANGING the switch state, call toggleProfileStatus function
	
    - TEXT for displaying profileType
      - USE userRef to get profileType from the "users"
        collection in the database.
		

END Component