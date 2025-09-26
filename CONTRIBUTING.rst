How To Contribute to NAIRR Community Resources Documentation

#.	Make edits in Codespaces (https://github.com/NAIRRProgram/nairr)

    #.	Click on 'Code'
    #.	Click on ‘Codespaces’ tab
    #.	Click on ‘Create new codespace on main’
    #.	If prompted to periodically git Fetch, click ‘Yes’. 

#.	Make changes

    #.	Docs are mostly in /docs/source/ - edit existing or create new files
    #.  For example, create 'mynewdoc.rst' as a sibling to 'index.rst'
    #.  In table of contents in 'index.rst', add new link to 'mynewdoc'
    #.  Add your content in 'mynewdoc.rst'

#.	Preview local build (builds automatically on changes)

    #.	Codespaces terminal bottom pane
    #.  cd docs
    #.  make html
    #.  python -mhttp.server, open in browser
    #.  After making additional edits, repeat make command and refresh browser to preview new changes

#.	Submit Pull Request

    #.	Click ’Github Pull request’ icon (Left-hand, bottom side of VSCode)
    #.	https://docs.github.com/en/codespaces/developing-in-a-codespace/using-github-codespaces-for-pull-requests
    #.	Select all modified files
    #.  Commit and Create a Pull Request
    #.  Would you like to stage and commit all your changes? Yes.
    #.  Committing to a protected branch and may not be able to merge changes.  Click "Create a new branch".
    #.  Add commit message, click check mark in top right to accept commit message
    #.	Add a meaningful title and description
    #.	Click 'Create pull request'
    #   Publish branch button

#.  (Optional) Final preview in Github (https://github.com/NAIRRProgram/nairr)
    #.  Click on 'Pull Requests' at top
    #.  Find your pull request and click on it
    #.  Once checks are successfull, click Show, and Details to see a full preview of ReadTheDocs with your new content.
    #.  Any comments/feedback will be posted here on your pull request.  If your changes are not showing up, you can check status here and comment.


