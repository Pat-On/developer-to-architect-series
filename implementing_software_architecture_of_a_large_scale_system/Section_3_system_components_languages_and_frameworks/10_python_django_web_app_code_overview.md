# Python Django Web App Code Overview

normal Python (Django) structure
- app - main code
  - static - static content - should go to the reversed proxy
        - we are going to serve it in future on reverse proxy
  - views + templates 