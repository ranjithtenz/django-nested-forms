Django Nested Forms
###################

Description
===========

I use Django for all my web projects.

I like to simplify my applications'forms by nesting them, for example when users can add many photos on a product.

This class will help you to easily create nested forms in your project. I created this for a big project, with many nested forms, and many users love it every days.

Requirements
============

Django 1.2+ (should be compatible with 1.1, but not tested)

Usage
=====

First of all, insert the nested_forms folder in in project's path.

In your forms.py
----------------

::

  from nested_forms import ComplexModelForm

Make you main form inherit from ComplexModelForm

In Meta subclass of your ComplexModelForm, add :

::

  formsets = {
  }

This dict has for keys the related objects' names.

For the examples form Django's tutorial, we could write :

::

  def get_choice_form(poll):
      class ChoiceForm(forms.ModelForm):
          class Meta:
              model = Choice
              exclude = ['poll']

          def save(self, commit=True):
              instance = super(ChoiceForm, self).save(commit=False)
              instance.poll = poll
              if commit:
                  instance.save()
                  self.save_m2m()
              return instance

      return ChoiceForm

  class PollForm(ComplexModelForm):
      class Meta:
          model = Poll
          formsets = {
              'choice_set': {
                   'form': lambda instance: get_choice_form(instance),
                   'extra': 1,
                   'initial': [
                       {
                           'choice': 'A sample choice',
                           'votes': 0,
                       }
                   ],
              },
          }


You now have a nested form. Let's start using it in your template.

In your templates
-----------------

Considering your nested form instance in a form variable in your context, you can access your access your form fields like any other forms. Your formsets are accessible by {{ form.formsets.[ name of related name of your object] }}. Don't forget to add the management form for each, and each formset is like any formset you would create manually.
*DON'T FORGET TO IMPLEMENT THE "ADD" TEMPLATE FILTER !*

::

  <form method="post">
      <fieldset>
          <legend>Poll</legend>
          {{ form.as_div }}

          {% with form.formsets.choice_set as choice_formset %}
              {{ choice_formset.management_form }}
              {% for choice_form in choice_formset.forms %}
              <fieldset>
              <legend>Choice {{ forloop.counter }}</legend>
              {{ choice_form.as_div }}
              </fieldset>
              {% endfor %}
              <button name="{{ choice_formset.prefix }}-TOTAL_FORMS" value="contact_formset.total_form_count|add:1">
                  Add a choice
              </button>
          {% endwith %}
      </fieldset>
      <input type="submit" value="Save poll"/>
  </form>

