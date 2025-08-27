var RevertChangeProcesso = Class.create();
RevertChangeProcesso.prototype = Object.extendsObject(AbstractAjaxProcessor, {

  revertToDraft: function () {
    var sysId   = this.getParameter('sysparm_sysid');
    var comment = (this.getParameter('sysparm_comment') || '').trim();

    var cr = new GlideRecord('change_request');
    if (!cr.get(sysId)) {
      return JSON.stringify({ status: 'error', message: 'Change Request not found.' });
    }

    var beforeLabel = cr.getDisplayValue('state');
    var TARGET_LABEL = 'New';
    var stateValue = this._getChoiceValue('change_request', 'state', TARGET_LABEL);
    if (!stateValue) {
      return JSON.stringify({ status: 'error', message: 'State "'+TARGET_LABEL+'" not found in sys_choice.' });
    }

    cr.setWorkflow(false);
    cr.setValue('state', stateValue);
    cr.work_notes = 'Sent back to ' + TARGET_LABEL + ': ' + comment;

    var updatedSysId = cr.update();
    var afterLabel   = cr.getDisplayValue('state');

    return JSON.stringify({
      status: updatedSysId ? 'success' : 'error',
      message: 'Moved from ' + beforeLabel + ' to ' + afterLabel + (updatedSysId ? ' (saved)' : ' (update failed)'),
      debug: { targetLabel: TARGET_LABEL, targetValue: stateValue, updatedSysId: String(updatedSysId) }
    });
  },

  _getChoiceValue: function (table, element, label) {
    var sc = new GlideRecord('sys_choice');
    sc.addQuery('name', table);
    sc.addQuery('element', element);
    sc.addQuery('label', label);
    sc.addQuery('inactive', false);
    sc.setLimit(1);
    sc.query();
    return sc.next() ? String(sc.value) : null;
  }
});
