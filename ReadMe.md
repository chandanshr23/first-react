# React From First🔥<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Healthcare Registration</title>
  <!-- Tailwind CSS (CDN) -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- jQuery (CDN) -->
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body class="bg-gray-50 text-gray-900">
  <div class="max-w-3xl mx-auto p-6">
    <h1 class="text-2xl font-semibold mb-4">Healthcare Registration</h1>
    <p class="text-sm text-gray-600 mb-6">Fill the form below. Fields marked required must be completed.</p>

    <form id="registrationForm" class="bg-white rounded-2xl shadow p-6 space-y-5">
      <!-- Dynamic fields render here -->
    </form>

    <div id="resultWrap" class="hidden mt-6">
      <h2 class="text-xl font-semibold mb-2">Submitted JSON</h2>
      <pre id="result" class="bg-gray-900 text-gray-100 p-4 rounded-xl overflow-auto"></pre>
    </div>
  </div>

  <script>
    // Your schema
    const formSchema = [
      { "title": "Full Name", "id": "fullName", "inputType": "text", "mandatory": true, "enable": true },
      { "title": "Age", "id": "age", "inputType": "number", "mandatory": true, "enable": true },
      { "title": "Gender", "id": "gender", "inputType": "select", "mandatory": true, "enable": true, "options": [
        { "text": "Male", "value": "male" }, { "text": "Female", "value": "female" }, { "text": "Other", "value": "other" }
      ]},
      { "title": "Phone Number", "id": "phone", "inputType": "tel", "charLength": 10, "mandatory": true, "enable": true },
      { "title": "Birth Date", "id": "birthDate", "inputType": "date", "mandatory": true, "enable": true },
      { "title": "Blood Group", "id": "bloodGroup", "inputType": "select", "mandatory": true, "enable": true, "options": [
        { "text": "A+", "value": "A+" }, { "text": "A-", "value": "A-" }, { "text": "B+", "value": "B+" }, { "text": "B-", "value": "B-" },
        { "text": "O+", "value": "O+" }, { "text": "O-", "value": "O-" }, { "text": "AB+", "value": "AB+" }, { "text": "AB-", "value": "AB-" }
      ]},
      { "title": "Height (cm)", "id": "height", "inputType": "number", "mandatory": false, "enable": true },
      { "title": "Weight (kg)", "id": "weight", "inputType": "number", "mandatory": false, "enable": true },
      { "title": "Smoking Habit", "id": "smoking", "inputType": "select", "mandatory": false, "enable": true, "options": [
        { "text": "Never", "value": "never" }, { "text": "Occasional", "value": "occasional" }, { "text": "Regular", "value": "regular" }
      ]},
      { "title": "Alcohol Consumption", "id": "alcohol", "inputType": "select", "mandatory": false, "enable": true, "options": [
        { "text": "Never", "value": "never" }, { "text": "Occasional", "value": "occasional" }, { "text": "Regular", "value": "regular" }
      ]},
      { "title": "Exercise Frequency", "id": "exercise", "inputType": "select", "mandatory": false, "enable": true, "options": [
        { "text": "Never", "value": "never" }, { "text": "1-2 times/week", "value": "1-2" },
        { "text": "3-5 times/week", "value": "3-5" }, { "text": "Daily", "value": "daily" }
      ]},
      { "title": "Accept Terms & Conditions", "id": "terms", "inputType": "checkbox", "mandatory": true, "enable": true }
    ];

    // Helpers
    function buildLabel(field) {
      const req = field.mandatory ? '<span class="text-red-600">*</span>' : '';
      return `<label for="${field.id}" class="block text-sm font-medium mb-1">${field.title} ${req}</label>`;
    }

    function buildInput(field) {
      const common = `id="${field.id}" name="${field.id}" ${field.enable ? '' : 'disabled'} ${field.mandatory ? 'required' : ''}`;
      const baseCls = "w-full rounded-xl border border-gray-300 focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 px-3 py-2 outline-none";
      const disabledCls = field.enable ? "" : "opacity-50 cursor-not-allowed";

      if (field.inputType === 'select') {
        const opts = (field.options || []).map(o => `<option value="${o.value}">${o.text}</option>`).join('');
        return `<select ${common} class="${baseCls} ${disabledCls}">
                  <option value="" disabled selected>Select ${field.title}</option>
                  ${opts}
                </select>`;
      }

      if (field.inputType === 'textarea') {
        return `<textarea ${common} rows="3" class="${baseCls} ${disabledCls}"></textarea>`;
      }

      if (field.inputType === 'checkbox') {
        return `<input type="checkbox" ${common} class="h-4 w-4 text-indigo-600 border-gray-300 rounded" />`;
      }

      // tel/number/date/text/email/password...
      const extra = [];
      if (field.inputType === 'tel' && field.charLength) {
        extra.push(`pattern="\\d{${field.charLength}}"`);
        extra.push(`inputmode="numeric"`);
        extra.push(`title="Enter ${field.charLength} digits"`);
        extra.push(`maxlength="${field.charLength}"`);
      }
      if (field.inputType === 'number') {
        extra.push('step="any"');
      }
      return `<input type="${field.inputType}" ${common} ${extra.join(' ')} class="${baseCls} ${disabledCls}">`;
    }

    function buildFieldRow(field) {
      if (field.inputType === 'checkbox' && field.id === 'terms') {
        return `
          <div class="flex items-start gap-3 p-4 bg-gray-50 rounded-xl border">
            ${buildInput(field)}
            <label for="${field.id}" class="text-sm">I agree to the <a href="#" class="text-indigo-600 underline">Terms &amp; Conditions</a>.</label>
          </div>
        `;
      }

      if (field.inputType === 'checkbox') {
        return `
          <div class="flex items-center gap-3">
            ${buildInput(field)}
            <label for="${field.id}" class="text-sm">${field.title}</label>
          </div>
        `;
      }

      return `
        <div>
          ${buildLabel(field)}
          ${buildInput(field)}
        </div>
      `;
    }

    function renderForm(schema) {
      const $form = $("#registrationForm");
      const rows = schema.map(buildFieldRow).join('');
      const actions = `
        <div class="pt-2 flex items-center gap-3">
          <button type="submit" class="px-5 py-2.5 rounded-xl bg-indigo-600 text-white hover:bg-indigo-700 focus:ring-2 focus:ring-indigo-500">Submit</button>
          <button type="button" id="resetBtn" class="px-5 py-2.5 rounded-xl border hover:bg-gray-50">Reset</button>
        </div>
      `;
      $form.html(rows + actions);
    }

    function collectData(schema) {
      const data = {};
      schema.forEach(f => {
        if (!f.enable) return;
        const $el = $(`#${f.id}`);
        if (f.inputType === 'checkbox') {
          data[f.id] = $el.is(':checked');
        } else {
          data[f.id] = ($el.val() ?? "").trim();
        }
      });
      return data;
    }

    function validatePhone($input, requiredLength) {
      const v = ($input.val() || "").trim();
      if (!v && $input.prop('required')) return false;
      if (!v) return true;
      return new RegExp(`^\\d{${requiredLength}}$`).test(v);
    }

    $(function() {
      renderForm(formSchema);

      $("#registrationForm").on("submit", function(e) {
        e.preventDefault();

        const $phone = $("#phone");
        const phoneField = formSchema.find(f => f.id === "phone");
        if (phoneField?.charLength && !validatePhone($phone, phoneField.charLength)) {
          $phone.addClass("ring-2 ring-red-500");
          $phone[0].setCustomValidity(`Please enter ${phoneField.charLength} digits`);
          $phone[0].reportValidity();
          setTimeout(() => $phone.removeClass("ring-2 ring-red-500"), 1500);
          return;
        } else if ($phone.length) {
          $phone[0].setCustomValidity("");
        }

        if (!this.checkValidity()) {
          this.reportValidity();
          return;
        }

        const data = collectData(formSchema);
        $("#result").text(JSON.stringify(data, null, 2));
        $("#resultWrap").removeClass("hidden");
        window.scrollTo({ top: document.body.scrollHeight, behavior: "smooth" });
      });

      $("#registrationForm").on("input change", "input, select, textarea", function() {
        $(this).removeClass("ring-2 ring-red-500");
        this.setCustomValidity("");
      });

      $("#registrationForm").on("click", "#resetBtn", function() {
        $("#registrationForm")[0].reset();
        $("#resultWrap").addClass("hidden");
        $("#result").text("");
      });
    });
  </script>
</body>
</html>
