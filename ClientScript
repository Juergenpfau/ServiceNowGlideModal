// Wrap everything in an IIFE so we don't leak variables into the global scope
(function () {

  // Expose a function to close the modal (called by your Cancel button)
  window.closeModal = function (e) {
    // If this came from a button in a <form>, stop the browser's default submit/navigation
    if (e && e.preventDefault) e.preventDefault();
    // Find the active Glide dialog and destroy it (closes the modal)
    GlideDialogWindow.get().destroy();
  };

  // Helper: show an inline error message under the textarea
  function showError(msg) {
    var el = document.getElementById('error_msg');
    if (!el) return;                // Guard if the element isn't present
    el.textContent = msg;           // Set the error text
    el.style.display = 'block';     // Make it visible
  }

  // Helper: hide and clear the error area
  function hideError() {
    var el = document.getElementById('error_msg');
    if (!el) return;                // Guard if the element isn't present
    el.style.display = 'none';      // Hide it
    el.textContent = '';            // Clear any previous message
  }

  // Expose the submit handler (called by your Submit button)
  window.submitRevert = function (e) {
    // Prevent the <g:ui_form> from doing a real form submit and navigating away
    if (e && e.preventDefault) e.preventDefault();

    // Start clean: hide any previous error
    hideError();

    // Read inputs from the DOM
    var sysId = (document.getElementById('sysparm_sys_id') || {}).value;            // record sys_id
    var comment = (document.getElementById('revert_comment') || {}).value.trim();   // user comment

    // Client-side validation: require a comment
    if (!comment) { showError('Comment is required.'); return; }
    // Defensive check: make sure the record id made it into the modal
    if (!sysId)   { showError('Missing record id.');   return; }

    // Prepare a GlideAjax call to your Script Include: RevertChangeProcessor.revertToDraft
    var ga = new GlideAjax('RevertChangeProcessor');
    ga.addParam('sysparm_name', 'revertToDraft');  // method name in the Script Include
    ga.addParam('sysparm_sysid', sysId);           // pass the record id
    ga.addParam('sysparm_comment', comment);       // pass the user's reason

    // Send the request; ServiceNow returns the answer as a string
    ga.getXMLAnswer(function (response) {
      try {
        // Prefer a JSON response contract: { status: 'success'|'error', message: '...' }
        var data = JSON.parse(response || '{}');

        if (data.status === 'success') {
          // Success path: notify the user
          alert(data.message || 'Change moved to Draft.');
          // Close the modal
          GlideDialogWindow.get().destroy();
          // Refresh the parent form to show the new state and work note
          if (top && top.g_form) top.g_form.reload(); else top.location.reload();
        } else {
          // Script Include ran but reported an error—surface the message inline
          showError(data.message || 'Server error.');
        }

      } catch (err) {
        // Fallback: if the Script Include returned plain text instead of JSON
        if ((response || '').toLowerCase().indexOf('success') !== -1) {
          // Treat any "success" text as OK and proceed
          alert('Change moved to Draft.');
          GlideDialogWindow.get().destroy();
          if (top && top.g_form) top.g_form.reload(); else top.location.reload();
        } else {
          // Unknown failure—show whatever came back
          showError(response || 'Unexpected error.');
        }
      }
    });
  };

  // Global key handler: stop Enter from submitting the <g:ui_form> while typing in the textarea
  document.addEventListener('keydown', function (evt) {
    // Only plain Enter (not Ctrl/Cmd+Enter)
    if (evt.key === 'Enter' && !evt.ctrlKey && !evt.metaKey) {
      var target = evt.target || {};
      // If focus is in the comment box, block the default submit
      if (target.id === 'revert_comment') {
        evt.preventDefault();
      }
    }
  });

  // When the modal content finishes loading, focus the textarea for faster input
  document.addEventListener('DOMContentLoaded', function () {
    var ta = document.getElementById('revert_comment');
    if (ta) ta.focus();
  });

})(); // End IIFE
